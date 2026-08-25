/*
  ============================================================================
  SMART DOOR LOCK  -  ESP32-CAM (AI Thinker)  +  SG90 Servo
  ----------------------------------------------------------------------------
  * Creates its OWN Wi-Fi hotspot (AP mode)  ->  no router / internet needed
  * Streams live MJPEG video to any phone/PC that joins the hotspot
  * Professional web UI with  OPEN GATE / KEEP LOCKED  controls
  * Drives an SG90 servo to physically lock / unlock the gate
  * Optional porch light (on-board flash GPIO) + auto re-lock timer + event log
  ----------------------------------------------------------------------------
  Upload:
    1. Board = "AI Thinker ESP32-CAM"  (ESP32 board package >= 2.0.0)
    2. Upload Speed = 115200
    3. Hold GPIO0 to GND, press RST, then Upload. Remove GPIO0 after.
  Use:
    Phone -> Wi-Fi -> "SmartDoorLock" / password "12345678" 
    Browser -> http://192.168.4.1
  ============================================================================
*/
#include "esp_camera.h"
#include "esp_http_server.h"
#include "driver/ledc.h"
#include "WiFi.h"
#include "soc/soc.h"
#include "soc/rtc_cntl_reg.h"
#include <string.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"

/* ============================ CONFIGURATION ============================== */
#define AP_SSID        "SmartDoorLock"     // Wi-Fi name shown on the phone
#define AP_PASSWORD    "12345678"          // min 8 chars (WPA2)
#define AP_CHANNEL     1

#define SERVO_PIN      2                   // AI Thinker free GPIO (2,4,12,13,14,15,33 ok)
#define OPEN_ANGLE     90                  // servo angle when UNLOCKED  (CALIBRATE!)
#define LOCK_ANGLE     0                   // servo angle when LOCKED    (CALIBRATE!)
#define SERVO_MIN_US   600                 // SG90 pulse @ 0 deg  (us)
#define SERVO_MAX_US   2400                // SG90 pulse @ 180 deg (us)
#define AUTO_RELOCK_MS 5000                // auto re-lock after open (0 = stay open)

#define FLASH_PIN      4                   // on-board white LED / external light
#define STATUS_LED_PIN 33                 // on-board red LED (indicator)

#define CAM_QUALITY    12                  // 1 (best) .. 63 (worst)  -> lower = heavier
#define CAM_FRAMESIZE FRAMESIZE_VGA        // VGA=640x480. Use SVGA/QVGA if it lags.
#define CAM_VFLIP      1                   // 1 = flip image vertically (fixes upside-down)
#define CAM_HMIRROR    1                   // 1 = mirror horizontally
#define STREAM_PORT    81                  // MJPEG stream on this port; controls stay on :80
/* ======================================================================== */

/* ---------------------------- CAMERA PINS ------------------------------- */
#define PWDN_GPIO_NUM     32
#define RESET_GPIO_NUM   -1
#define XCLK_GPIO_NUM     0
#define SIOD_GPIO_NUM    26
#define SIOC_GPIO_NUM    27
#define Y9_GPIO_NUM      35
#define Y8_GPIO_NUM      34
#define Y7_GPIO_NUM      39
#define Y6_GPIO_NUM      36
#define Y5_GPIO_NUM      21
#define Y4_GPIO_NUM      19
#define Y3_GPIO_NUM      18
#define Y2_GPIO_NUM       5
#define VSYNC_GPIO_NUM   25
#define HREF_GPIO_NUM    23
#define PCLK_GPIO_NUM    22
/* ------------------------------------------------------------------------ */

/* ------------------------------ GLOBALS --------------------------------- */
httpd_handle_t  stream_httpd = NULL;
WiFiServer       streamServer(STREAM_PORT);
TaskHandle_t     streamTaskHandle = NULL;
bool            gateOpen   = false;
unsigned long   unlockStart = 0;
bool            flashOn    = false;
char            apIPStr[16] = "192.168.4.1";

struct Evt { unsigned long t; char m[48]; };
Evt   events[24];
int   evtCount = 0;

/* ======================================================================== */
/*  SERVO (LED PWM via dedicated timer 1 so it never touches the camera's   */
/*  XCLK timer 0)                                                           */
/* ======================================================================== */
void setupServo() {
  ledc_timer_config_t t = {
    .speed_mode      = LEDC_LOW_SPEED_MODE,
    .duty_resolution = LEDC_TIMER_16_BIT,
    .timer_num       = LEDC_TIMER_1,
    .freq_hz         = 50,
    .clk_cfg         = LEDC_AUTO_CLK
  };
  ledc_timer_config(&t);

  ledc_channel_config_t ch = {
    .gpio_num   = (gpio_num_t)SERVO_PIN,
    .speed_mode = LEDC_LOW_SPEED_MODE,
    .channel    = LEDC_CHANNEL_1,
    .timer_sel  = LEDC_TIMER_1,
    .duty       = 0,
    .hpoint     = 0
  };
  ledc_channel_config(&ch);
  setServoAngle(LOCK_ANGLE);
}

void setServoAngle(int angle) {
  angle = constrain(angle, 0, 180);
  long us = map(angle, 0, 180, SERVO_MIN_US, SERVO_MAX_US);
  uint32_t duty = (uint32_t)((float)us / 20000.0f * 65535.0f);
  ledc_set_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_1, duty);
  ledc_update_duty(LEDC_LOW_SPEED_MODE, LEDC_CHANNEL_1);
}

void openGate() {
  gateOpen = true;
  unlockStart = millis();
  setServoAngle(OPEN_ANGLE);
  digitalWrite(STATUS_LED_PIN, HIGH);
  addEvent("Gate OPENED (remote)");
}
void lockGate() {
  gateOpen = false;
  setServoAngle(LOCK_ANGLE);
  digitalWrite(STATUS_LED_PIN, LOW);
  addEvent("Gate LOCKED");
}

void addEvent(const char* msg) {
  if (evtCount < 24) {
    events[evtCount].t = millis() / 1000;
    strncpy(events[evtCount].m, msg, 47); events[evtCount].m[47] = 0;
    evtCount++;
  } else {
    for (int i = 0; i < 23; i++) events[i] = events[i + 1];
    events[23].t = millis() / 1000;
    strncpy(events[23].m, msg, 47); events[23].m[47] = 0;
  }
}

/* ======================================================================== */
/*  CAMERA                                                                  */
/* ======================================================================== */
bool initCamera() {
  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer   = LEDC_TIMER_0;
  config.pin_d0  = Y2_GPIO_NUM;
  config.pin_d1  = Y3_GPIO_NUM;
  config.pin_d2  = Y4_GPIO_NUM;
  config.pin_d3  = Y5_GPIO_NUM;
  config.pin_d4  = Y6_GPIO_NUM;
  config.pin_d5  = Y7_GPIO_NUM;
  config.pin_d6  = Y8_GPIO_NUM;
  config.pin_d7  = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href  = HREF_GPIO_NUM;
  config.pin_sccb_sda = SIOD_GPIO_NUM;
  config.pin_sccb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.pixel_format = PIXFORMAT_JPEG;

  if (psramFound()) {
    config.frame_size  = CAM_FRAMESIZE;
    config.jpeg_quality = CAM_QUALITY;
    config.fb_count    = 2;
  } else {
    config.frame_size  = FRAMESIZE_CIF;
    config.jpeg_quality = CAM_QUALITY;
    config.fb_count    = 1;
  }
  config.grab_mode = CAMERA_GRAB_LATEST;

  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x\n", err);
    return false;
  }
  sensor_t* s = esp_camera_sensor_get();
  if (s) {
    s->set_brightness(s, 1);
    s->set_contrast(s, 0);
    s->set_saturation(s, 0);
    s->set_vflip(s, CAM_VFLIP);
    s->set_hmirror(s, CAM_HMIRROR);
  }
  return true;
}

/* ======================================================================== */
/*  WEB UI (professional, fully inline, mobile-friendly)                    */
/* ======================================================================== */
static const char INDEX_HTML[] = R"SDL(
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0b1020">
<title>Smart Door Lock</title>
<style>
  :root{
    --bg:#0b1020; --card:rgba(255,255,255,.06); --line:rgba(255,255,255,.10);
    --accent:#4f8cff; --accent2:#22d3ee; --open:#22c55e; --lock:#ef4444;
    --text:#e7ecf5; --muted:#9aa7bd;
  }
  *{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent}
  body{font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Helvetica,Arial,sans-serif;
    background:radial-gradient(1200px 800px at 85% -10%, #1b2547 0%, #0b1020 55%);
    color:var(--text);min-height:100vh;padding:14px}
  .app{max-width:980px;margin:0 auto}
  header{display:flex;align-items:center;justify-content:space-between;gap:12px;
    padding:14px 18px;background:var(--card);border:1px solid var(--line);
    border-radius:16px;backdrop-filter:blur(10px);margin-bottom:14px}
  .brand{display:flex;align-items:center;gap:12px}
  .logo{width:44px;height:44px;border-radius:13px;display:grid;place-items:center;
    background:linear-gradient(135deg,var(--accent),var(--accent2));
    box-shadow:0 8px 24px rgba(79,140,255,.35)}
  .brand h1{font-size:18px;font-weight:700;letter-spacing:.3px}
  .brand p{font-size:12px;color:var(--muted)}
  .conn{display:flex;gap:16px;align-items:center;font-size:13px;color:var(--muted);text-align:right}
  .conn b{color:var(--text)}
  .grid{display:grid;grid-template-columns:1fr;gap:14px}
  @media(min-width:820px){.grid{grid-template-columns:1.6fr 1fr}}
  .video-card{position:relative;background:#000;border-radius:18px;overflow:hidden;
    border:1px solid var(--line);box-shadow:0 20px 50px rgba(0,0,0,.45)}
  #stream{width:100%;aspect-ratio:4/3;object-fit:cover;display:block;background:#000}
  .badges{position:absolute;top:12px;left:12px;display:flex;gap:8px}
  .badge{padding:6px 10px;border-radius:999px;font-size:12px;font-weight:700;
    backdrop-filter:blur(6px);border:1px solid transparent}
  .badge.live{background:rgba(239,68,68,.18);color:#ff9b9b;border-color:rgba(239,68,68,.4)}
  .badge.locked{background:rgba(239,68,68,.18);color:#ff9b9b;border-color:rgba(239,68,68,.4)}
  .badge.unlocked{background:rgba(34,197,94,.18);color:#86efac;border-color:rgba(34,197,94,.4)}
  .timer{position:absolute;top:12px;right:12px;background:rgba(0,0,0,.55);
    padding:6px 12px;border-radius:10px;font-size:13px;font-weight:700;color:#fde68a}
  .scan{position:absolute;inset:0;display:grid;place-items:center;color:var(--muted);
    font-size:14px;background:#000}
  .controls{background:var(--card);border:1px solid var(--line);border-radius:18px;
    padding:18px;display:flex;flex-direction:column;gap:12px;backdrop-filter:blur(10px)}
  .btn{border:none;border-radius:14px;padding:16px;font-size:16px;font-weight:700;color:#fff;
    cursor:pointer;display:flex;align-items:center;justify-content:center;gap:10px;
    transition:transform .08s ease, filter .2s ease}
  .btn:active{transform:scale(.98)}
  .btn.open{background:linear-gradient(135deg,#16a34a,#22c55e);box-shadow:0 10px 30px rgba(34,197,94,.35)}
  .btn.lock{background:linear-gradient(135deg,#b91c1c,#ef4444);box-shadow:0 10px 30px rgba(239,68,68,.3)}
  .btn.ghost{background:linear-gradient(135deg,#334155,#475569)}
  .btn:disabled{opacity:.55;cursor:not-allowed;filter:grayscale(.4)}
  .meta{font-size:13px;color:var(--muted);display:flex;justify-content:space-between;padding:0 4px}
  .meta b{color:var(--text)}
  .log-card{background:var(--card);border:1px solid var(--line);border-radius:18px;
    padding:16px;backdrop-filter:blur(10px);grid-column:1/-1}
  .log-card h3{font-size:12px;margin-bottom:10px;color:var(--muted);
    text-transform:uppercase;letter-spacing:.08em}
  #log{list-style:none;max-height:210px;overflow:auto;display:flex;flex-direction:column;gap:8px}
  #log li{display:flex;gap:10px;font-size:13px;padding:8px 10px;background:rgba(255,255,255,.04);border-radius:10px}
  #log .t{color:var(--muted);font-variant-numeric:tabular-nums;min-width:56px}
  footer{text-align:center;color:var(--muted);font-size:12px;margin-top:16px;line-height:1.6}
  .toast{position:fixed;bottom:22px;left:50%;transform:translateX(-50%) translateY(10px);
    background:#111827;border:1px solid var(--line);padding:12px 18px;border-radius:12px;
    font-size:14px;opacity:0;transition:opacity .25s, transform .25s;pointer-events:none;z-index:9}
  .toast.show{opacity:1;transform:translateX(-50%) translateY(0)}
</style>
</head>
<body>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="11" width="18" height="10" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/>
          </svg>
        </div>
        <div>
          <h1>Smart Door Lock</h1>
          <p>ESP32-CAM &middot; Live Secure Feed</p>
        </div>
      </div>
      <div class="conn">
        <div>Viewers<br><b id="clients">0</b></div>
        <div>Uptime<br><b id="uptime">0m 0s</b></div>
      </div>
    </header>

    <div class="grid">
      <section class="video-card">
        <div class="badges">
          <span class="badge live">&#9679; LIVE</span>
          <span id="gateBadge" class="badge locked">LOCKED</span>
        </div>
        <div id="timer" class="timer" style="display:none"></div>
        <div id="scan" class="scan">Connecting to camera&hellip;</div>
        <img id="stream" alt="camera stream" onload="document.getElementById('scan').style.display='none'">
      </section>

      <section class="controls">
        <button id="openBtn" class="btn open">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="10" rx="2"/><path d="M7 11V7a5 5 0 0 1 9.9-1"/></svg>
          Open Gate
        </button>
        <button id="lockBtn" class="btn lock">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="10" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
          Keep Locked
        </button>
        <button id="flashBtn" class="btn ghost">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M13 2 3 14h7l-1 8 10-12h-7l1-8z"/></svg>
          Light: OFF
        </button>
        <a class="btn ghost" href="/snapshot" target="_blank" style="text-decoration:none">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"/><circle cx="12" cy="13" r="4"/></svg>
          Snapshot
        </a>
        <div class="meta"><span>Auto re-lock</span><b id="autoVal">5s</b></div>
        <div class="meta"><span>Network</span><b>SmartDoorLock</b></div>
      </section>

      <section class="log-card">
        <h3>Activity Log</h3>
        <ul id="log"></ul>
      </section>
    </div>

    <footer>
      Join Wi-Fi <b>SmartDoorLock</b> &middot; password <b>12345678</b><br>
      Then open <b>http://192.168.4.1</b> on your phone browser.
    </footer>
  </div>

  <div id="toast" class="toast"></div>

<script>
  const gateBadge = document.getElementById('gateBadge');
  const openBtn   = document.getElementById('openBtn');
  const lockBtn   = document.getElementById('lockBtn');
  const flashBtn  = document.getElementById('flashBtn');
  const timerEl   = document.getElementById('timer');
  const logEl     = document.getElementById('log');
  const clientsEl = document.getElementById('clients');
  const uptimeEl  = document.getElementById('uptime');
  const autoVal   = document.getElementById('autoVal');

  // Stream is served on a SEPARATE port so the control buttons never get blocked.
  document.getElementById('stream').src =
    location.protocol + '//' + location.hostname + ':81/stream';

  function fmtUptime(s){
    s = Math.floor(s);
    const h = Math.floor(s/3600), m = Math.floor((s%3600)/60), ss = s%60;
    return (h? h+'h ':'') + m + 'm ' + ss + 's';
  }
  let toastT;
  function toast(msg){
    const el = document.getElementById('toast');
    el.textContent = msg; el.classList.add('show');
    clearTimeout(toastT); toastT = setTimeout(()=>el.classList.remove('show'), 2200);
  }
  async function post(url){
    openBtn.disabled = lockBtn.disabled = true;
    try{
      const r = await fetch(url, {cache:'no-store'});
      const d = await r.json();
      toast(d.message || 'OK');
      update(d);
    }catch(e){ toast('Request failed - is the camera near?'); }
    finally{ openBtn.disabled = lockBtn.disabled = false; }
  }
  openBtn.onclick = () => post('/open');
  lockBtn.onclick = () => post('/lock');
  flashBtn.onclick = async () => {
    try{
      const r = await fetch('/flash', {cache:'no-store'});
      const d = await r.json();
      flashBtn.innerHTML = flashBtn.innerHTML.replace(/Light: (ON|OFF)/, 'Light: ' + (d.on?'ON':'OFF'));
      toast(d.on ? 'Light ON' : 'Light OFF');
    }catch(e){}
  };

  function update(d){
    gateBadge.textContent = (d.gate === 'unlocked') ? 'UNLOCKED' : 'LOCKED';
    gateBadge.className = 'badge ' + ((d.gate === 'unlocked') ? 'unlocked' : 'locked');
    clientsEl.textContent = d.clients;
    uptimeEl.textContent  = fmtUptime(d.uptime);
    autoVal.textContent   = d.autoRelock ? (d.autoSec + 's') : 'OFF';
    if (d.gate === 'unlocked' && d.autoRelock && d.unlockRemaining > 0){
      timerEl.textContent = 'Auto-lock in ' + Math.ceil(d.unlockRemaining/1000) + 's';
      timerEl.style.display = 'block';
    } else {
      timerEl.style.display = 'none';
    }
    if (d.events) renderLog(d.events);
  }
  function renderLog(ev){
    logEl.innerHTML = '';
    ev.slice().reverse().forEach(e => {
      const li = document.createElement('li');
      const t  = document.createElement('span'); t.className = 't'; t.textContent = fmtUptime(e.t);
      const m  = document.createElement('span'); m.textContent = e.m;
      li.appendChild(t); li.appendChild(m); logEl.appendChild(li);
    });
  }
  async function poll(){
    try{
      const r = await fetch('/status', {cache:'no-store'});
      const d = await r.json();
      update(d);
    }catch(e){}
  }
  setInterval(poll, 1500);
  poll();
</script>
</body>
</html>
)SDL";

/* ======================================================================== */
/*  HTTP HANDLERS                                                            */
/* ======================================================================== */
static esp_err_t index_handler(httpd_req_t *req) {
  httpd_resp_set_type(req, "text/html; charset=UTF-8");
  httpd_resp_set_hdr(req, "Cache-Control", "no-cache");
  return httpd_resp_send(req, INDEX_HTML, strlen(INDEX_HTML));
}

static esp_err_t status_handler(httpd_req_t *req) {
  char json[1200];
  unsigned long remain = 0;
  if (gateOpen && AUTO_RELOCK_MS > 0) {
    long r = (long)AUTO_RELOCK_MS - (long)(millis() - unlockStart);
    remain = (r > 0) ? (unsigned long)r : 0;
  }
  int n = snprintf(json, sizeof(json),
    "{\"gate\":\"%s\",\"autoRelock\":%s,\"autoSec\":%lu,\"unlockRemaining\":%lu,"
    "\"clients\":%d,\"uptime\":%lu,\"ip\":\"%s\",\"ap\":\"%s\",\"flash\":%s,\"events\":[",
    gateOpen ? "unlocked" : "locked",
    AUTO_RELOCK_MS > 0 ? "true" : "false",
    (unsigned long)(AUTO_RELOCK_MS / 1000),
    remain,
    (int)WiFi.softAPgetStationNum(),
    millis() / 1000,
    apIPStr,
    AP_SSID,
    flashOn ? "true" : "false");

  for (int i = 0; i < evtCount; i++) {
    n += snprintf(json + n, sizeof(json) - n, "%s{\"t\":%lu,\"m\":\"%s\"}",
                  i ? "," : "", events[i].t, events[i].m);
  }
  n += snprintf(json + n, sizeof(json) - n, "]}");
  httpd_resp_set_type(req, "application/json");
  httpd_resp_set_hdr(req, "Cache-Control", "no-cache");
  return httpd_resp_send(req, json, n);
}

static esp_err_t open_handler(httpd_req_t *req) {
  openGate();
  char buf[128];
  snprintf(buf, sizeof(buf), "{\"ok\":true,\"gate\":\"unlocked\",\"message\":\"Gate opened\"}");
  httpd_resp_set_type(req, "application/json");
  return httpd_resp_send(req, buf, strlen(buf));
}

static esp_err_t lock_handler(httpd_req_t *req) {
  lockGate();
  char buf[128];
  snprintf(buf, sizeof(buf), "{\"ok\":true,\"gate\":\"locked\",\"message\":\"Gate locked\"}");
  httpd_resp_set_type(req, "application/json");
  return httpd_resp_send(req, buf, strlen(buf));
}

static esp_err_t flash_handler(httpd_req_t *req) {
  flashOn = !flashOn;
  digitalWrite(FLASH_PIN, flashOn ? HIGH : LOW);
  char buf[128];
  snprintf(buf, sizeof(buf), "{\"ok\":true,\"on\":%s,\"message\":\"Light %s\"}",
           flashOn ? "true" : "false", flashOn ? "ON" : "OFF");
  httpd_resp_set_type(req, "application/json");
  return httpd_resp_send(req, buf, strlen(buf));
}

static esp_err_t snapshot_handler(httpd_req_t *req) {
  camera_fb_t *fb = esp_camera_fb_get();
  if (!fb) { httpd_resp_send_500(req); return ESP_FAIL; }
  httpd_resp_set_type(req, "image/jpeg");
  httpd_resp_set_hdr(req, "Content-Disposition", "inline; filename=capture.jpg");
  esp_err_t r = httpd_resp_send(req, (const char *)fb->buf, fb->len);
  esp_camera_fb_return(fb);
  return r;
}

/* Stream runs in its OWN task on port STREAM_PORT so it NEVER blocks the
   control web server (port 80). This is what makes the buttons/light work
   while the video is live. */
void streamTask(void * pvParameters) {
  for (;;) {
    WiFiClient client = streamServer.available();
    if (client) {
      Serial.println("Stream client connected");
      client.print("HTTP/1.1 200 OK\r\n");
      client.print("Content-Type: multipart/x-mixed-replace; boundary=frame\r\n\r\n");
      while (client.connected()) {
        camera_fb_t *fb = esp_camera_fb_get();
        if (!fb) { delay(10); continue; }
        if (fb->format != PIXFORMAT_JPEG) {   // we configure JPEG, but guard anyway
          esp_camera_fb_return(fb);
          delay(10);
          continue;
        }
        client.print("--frame\r\n");
        client.print("Content-Type: image/jpeg\r\n");
        client.print("Content-Length: ");
        client.print(fb->len);
        client.print("\r\n\r\n");
        client.write(fb->buf, fb->len);
        client.print("\r\n");
        esp_camera_fb_return(fb);
        delay(20);   // throttle + yield to other tasks
      }
      client.stop();
      Serial.println("Stream client disconnected");
    }
    delay(20);
  }
}

void startCameraServer() {
  httpd_config_t config = HTTPD_DEFAULT_CONFIG();
  config.server_port    = 80;
  config.max_uri_handlers = 8;
  if (config.server_port + config.ctrl_port > 65535) config.ctrl_port = 32767;

  if (httpd_start(&stream_httpd, &config) != ESP_OK) {
    Serial.println("Failed to start web server");
    return;
  }
  httpd_uri_t index_uri = {
    .uri = "/", .method = HTTP_GET, .handler = index_handler, .user_ctx = NULL
  };
  httpd_uri_t status_uri = {
    .uri = "/status", .method = HTTP_GET, .handler = status_handler, .user_ctx = NULL
  };
  httpd_uri_t open_uri = {
    .uri = "/open", .method = HTTP_GET, .handler = open_handler, .user_ctx = NULL
  };
  httpd_uri_t lock_uri = {
    .uri = "/lock", .method = HTTP_GET, .handler = lock_handler, .user_ctx = NULL
  };
  httpd_uri_t flash_uri = {
    .uri = "/flash", .method = HTTP_GET, .handler = flash_handler, .user_ctx = NULL
  };
  httpd_uri_t snap_uri = {
    .uri = "/snapshot", .method = HTTP_GET, .handler = snapshot_handler, .user_ctx = NULL
  };
  httpd_register_uri_handler(stream_httpd, &index_uri);
  httpd_register_uri_handler(stream_httpd, &status_uri);
  httpd_register_uri_handler(stream_httpd, &open_uri);
  httpd_register_uri_handler(stream_httpd, &lock_uri);
  httpd_register_uri_handler(stream_httpd, &flash_uri);
  httpd_register_uri_handler(stream_httpd, &snap_uri);
}

/* ======================================================================== */
/*  SETUP / LOOP                                                             */
/* ======================================================================== */
void setup() {
  WRITE_PERI_REG(RTC_CNTL_BROWN_OUT_REG, 0); // disable brownout detector
  Serial.begin(115200);
  Serial.setDebugOutput(false);

  pinMode(FLASH_PIN, OUTPUT);       digitalWrite(FLASH_PIN, LOW);
  pinMode(STATUS_LED_PIN, OUTPUT);  digitalWrite(STATUS_LED_PIN, LOW);

  setupServo();

  if (!initCamera()) {
    Serial.println("Camera init failed - check wiring / board selection");
    addEvent("Camera init FAILED");
  } else {
    addEvent("Camera ready");
  }

  WiFi.mode(WIFI_AP);
  WiFi.softAP(AP_SSID, AP_PASSWORD, AP_CHANNEL);
  WiFi.setSleep(false);
  IPAddress ip = WiFi.softAPIP();
  strncpy(apIPStr, ip.toString().c_str(), 15);
  Serial.print("AP IP: "); Serial.println(ip);

  addEvent("Hotspot up");
  startCameraServer();

  // Start the MJPEG stream on its OWN port/task so controls stay responsive.
  streamServer.begin();
  xTaskCreatePinnedToCore(streamTask, "streamTask", 10240, NULL, 2, &streamTaskHandle, 0);
  addEvent("Stream ready :81");
}

void loop() {
  if (gateOpen && AUTO_RELOCK_MS > 0 && (millis() - unlockStart) >= AUTO_RELOCK_MS) {
    lockGate();
    addEvent("Auto re-locked");
  }
  delay(100);
}
