<!DOCTYPE html>
<html lang="en" dir="ltr">

<head>

  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1"
  >

  <title>Voice note</title>

  <style>

    * {
      box-sizing: border-box;
    }

    html,
    body {
      height: 100%;
      margin: 0;
    }

    body {
      font-family: Arial, Helvetica, sans-serif;
      background: #f3f4f6;
      color: #111827;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 16px;
    }

    .card {
      width: 100%;
      max-width: 340px;
      padding: 22px 18px;
      background: white;
      border-radius: 14px;
      box-shadow: 0 6px 24px rgba(0,0,0,.12);
      text-align: center;
    }

    h1 {
      margin: 0 0 4px;
      font-size: 17px;
    }

    .hint {
      min-height: 34px;
      margin: 0 0 14px;
      font-size: 12px;
      color: #6b7280;
      line-height: 1.4;
    }

    .hint.error {
      color: #b91c1c;
    }

    .timer {
      margin: 6px 0 16px;
      font-size: 34px;
      font-weight: 700;
      font-variant-numeric: tabular-nums;
    }

    .rec-btn {
      width: 84px;
      height: 84px;
      border: 0;
      border-radius: 50%;
      background: #dc2626;
      color: white;
      font-size: 34px;
      cursor: pointer;
      box-shadow: 0 4px 14px rgba(220,38,38,.35);
    }

    .rec-btn.recording {
      animation: recPulse 1.1s infinite;
    }

    @keyframes recPulse {
      0%, 100% { box-shadow: 0 0 0 0 rgba(220,38,38,.55); }
      50%      { box-shadow: 0 0 0 16px rgba(220,38,38,0); }
    }

    .row {
      display: flex;
      gap: 8px;
      justify-content: center;
      margin-top: 14px;
    }

    .btn {
      flex: 1;
      height: 42px;
      border: 0;
      border-radius: 9px;
      font-weight: 700;
      font-size: 13px;
      cursor: pointer;
    }

    .btn.send {
      background: #2563eb;
      color: white;
    }

    .btn.emergency {
      background: #dc2626;
      color: white;
    }

    .btn.secondary {
      background: #f3f4f6;
      color: #111827;
    }

    .btn:disabled {
      opacity: .55;
      cursor: default;
    }

    audio {
      width: 100%;
      margin-top: 4px;
    }

    [hidden] {
      display: none !important;
    }

  </style>

</head>


<body>

  <div class="card">

    <h1 id="title">Voice note</h1>

    <p id="hint" class="hint"></p>


    <!-- STEP 1 + 2: record -->

    <div id="recordView">

      <div id="timer" class="timer">0:00</div>

      <button
        type="button"
        id="recBtn"
        class="rec-btn"
        onclick="toggleRecording()">
        🎤
      </button>

    </div>


    <!-- STEP 3: review + send -->

    <div id="reviewView" hidden>

      <audio id="preview" controls></audio>

      <div class="row">

        <button
          type="button"
          id="sendBtn"
          class="btn send"
          onclick="sendVoice('normal')">
          Send
        </button>

        <button
          type="button"
          id="emergencyBtn"
          class="btn emergency"
          onclick="sendVoice('emergency')">
          🚨 Emergency
        </button>

      </div>

      <div class="row">

        <button
          type="button"
          id="againBtn"
          class="btn secondary"
          onclick="recordAgain()">
          Record again
        </button>

      </div>

    </div>

  </div>


  <script>

    var MAX_SECONDS = 300;   // 5 minutes

    var strings = {

      en: {
        title: "Voice note",
        tapToRecord: "Tap the microphone to start recording.",
        recording: "Recording... tap again to stop.",
        review: "Listen, then send it to the chat.",
        send: "Send",
        emergency: "🚨 Emergency",
        again: "Record again",
        sent: "Sent ✓",
        denied: "Microphone access was blocked. Allow the microphone for this page in your browser, then try again.",
        unsupported: "This browser can't record audio.",
        noOpener: "Open this page from the chat (🎤 button) so the recording can be sent.",
        sendFailed: "Could not send. Close this window and try again from the chat."
      },

      ar: {
        title: "رسالة صوتية",
        tapToRecord: "اضغط على الميكروفون لبدء التسجيل.",
        recording: "جاري التسجيل... اضغط مرة أخرى للإيقاف.",
        review: "استمع للتسجيل ثم أرسله إلى الشات.",
        send: "إرسال",
        emergency: "🚨 طوارئ",
        again: "تسجيل من جديد",
        sent: "تم الإرسال ✓",
        denied: "تم حظر الوصول إلى الميكروفون. اسمح بالميكروفون لهذه الصفحة من إعدادات المتصفح ثم حاول مرة أخرى.",
        unsupported: "هذا المتصفح لا يدعم تسجيل الصوت.",
        noOpener: "افتح هذه الصفحة من زر 🎤 داخل الشات حتى يمكن إرسال التسجيل.",
        sendFailed: "تعذر الإرسال. أغلق هذه النافذة وحاول مرة أخرى من الشات."
      }

    };


    var params = new URLSearchParams(window.location.search);

    var lang = params.get("lang") === "ar" ? "ar" : "en";

    var S = strings[lang];

    document.documentElement.lang = lang;
    document.documentElement.dir = lang === "ar" ? "rtl" : "ltr";


    var hint = document.getElementById("hint");
    var timerEl = document.getElementById("timer");
    var recBtn = document.getElementById("recBtn");
    var recordView = document.getElementById("recordView");
    var reviewView = document.getElementById("reviewView");
    var preview = document.getElementById("preview");
    var sendBtn = document.getElementById("sendBtn");
    var emergencyBtn = document.getElementById("emergencyBtn");
    var againBtn = document.getElementById("againBtn");

    document.getElementById("title").textContent = S.title;
    document.title = S.title;
    sendBtn.textContent = S.send;
    emergencyBtn.textContent = S.emergency;
    againBtn.textContent = S.again;


    var stream = null;
    var recorder = null;
    var chunks = [];
    var startedAt = 0;
    var tickTimer = null;
    var recordedBlob = null;
    var recordedSeconds = 0;
    var previewUrl = null;


    function setHint(text, isError) {

      hint.textContent = text || "";

      hint.className = "hint" + (isError ? " error" : "");

    }


    function formatTime(sec) {

      var m = Math.floor(sec / 60);
      var s = sec % 60;

      return m + ":" + (s < 10 ? "0" : "") + s;

    }


    function pickMimeType() {

      var candidates = [
        "audio/webm;codecs=opus",
        "audio/webm",
        "audio/mp4",
        "audio/ogg;codecs=opus"
      ];

      if (!window.MediaRecorder || !MediaRecorder.isTypeSupported) {
        return "";
      }

      for (var i = 0; i < candidates.length; i++) {

        if (MediaRecorder.isTypeSupported(candidates[i])) {
          return candidates[i];
        }

      }

      return "";

    }


    function stopStream() {

      if (stream) {

        stream.getTracks().forEach(function(track) {
          track.stop();
        });

        stream = null;

      }

    }


    function toggleRecording() {

      if (recorder && recorder.state === "recording") {

        stopRecording();

      } else {

        startRecording();

      }

    }


    function startRecording() {

      if (
        !navigator.mediaDevices ||
        !navigator.mediaDevices.getUserMedia ||
        !window.MediaRecorder
      ) {

        setHint(S.unsupported, true);

        return;

      }

      setHint("");

      navigator.mediaDevices.getUserMedia({ audio: true })

        .then(function(s) {

          stream = s;

          var mime = pickMimeType();

          recorder =
            mime
              ? new MediaRecorder(stream, { mimeType: mime })
              : new MediaRecorder(stream);

          chunks = [];

          recorder.addEventListener("dataavailable", function(e) {

            if (e.data && e.data.size > 0) {
              chunks.push(e.data);
            }

          });

          recorder.addEventListener("stop", onRecorderStopped);

          recorder.start();

          startedAt = Date.now();

          recBtn.classList.add("recording");

          recBtn.textContent = "⏹";

          setHint(S.recording);

          tickTimer = setInterval(function() {

            var elapsed = Math.floor((Date.now() - startedAt) / 1000);

            timerEl.textContent = formatTime(elapsed);

            if (elapsed >= MAX_SECONDS) {
              stopRecording();
            }

          }, 250);

        })

        .catch(function() {

          setHint(S.denied, true);

        });

    }


    function stopRecording() {

      if (recorder && recorder.state === "recording") {
        recorder.stop();
      }

    }


    function onRecorderStopped() {

      clearInterval(tickTimer);

      recordedSeconds = Math.max(1, Math.round((Date.now() - startedAt) / 1000));

      stopStream();

      recBtn.classList.remove("recording");

      recBtn.textContent = "🎤";

      var type = (recorder && recorder.mimeType) || "audio/webm";

      recordedBlob = new Blob(chunks, { type: type });

      if (previewUrl) {
        URL.revokeObjectURL(previewUrl);
      }

      previewUrl = URL.createObjectURL(recordedBlob);

      preview.src = previewUrl;

      recordView.hidden = true;

      reviewView.hidden = false;

      setHint(S.review);

    }


    function recordAgain() {

      recordedBlob = null;

      preview.removeAttribute("src");

      timerEl.textContent = "0:00";

      reviewView.hidden = true;

      recordView.hidden = false;

      setHint(S.tapToRecord);

    }


    function sendVoice(kind) {

      if (!recordedBlob) {
        return;
      }

      if (!window.opener || window.opener.closed) {

        setHint(S.noOpener, true);

        return;

      }

      sendBtn.disabled = true;
      emergencyBtn.disabled = true;
      againBtn.disabled = true;

      var reader = new FileReader();

      reader.onload = function() {

        try {

          window.opener.postMessage(
            {
              type: "CHAT_VOICE_NOTE",
              kind: kind,
              mimeType: (recordedBlob.type || "audio/webm").split(";")[0],
              duration: recordedSeconds,
              dataUrl: String(reader.result || "")
            },
            "*"
          );

          setHint(S.sent);

          setTimeout(function() {
            window.close();
          }, 500);

        } catch (e) {

          setHint(S.sendFailed, true);

          sendBtn.disabled = false;
          emergencyBtn.disabled = false;
          againBtn.disabled = false;

        }

      };

      reader.onerror = function() {

        setHint(S.sendFailed, true);

        sendBtn.disabled = false;
        emergencyBtn.disabled = false;
        againBtn.disabled = false;

      };

      reader.readAsDataURL(recordedBlob);

    }


    window.addEventListener("beforeunload", stopStream);


    /* start */

    if (!window.opener) {
      setHint(S.noOpener, true);
    } else {
      setHint(S.tapToRecord);
    }

  </script>

</body>

</html>
