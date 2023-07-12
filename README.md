# task-2-bahart-intern
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8" />
    <title>Video Conferencing using RTCMultiConnection</title>
    <meta
      name="viewport"
      content="width=device-width, initial-scale=1.0, minimum-scale=1.0"
    />
    <link rel="shortcut icon" href="https://muazkhan.com:9001/demos/logo.png" />
    <link
      rel="stylesheet"
      href="https://muazkhan.com:9001/demos/stylesheet.css"
    />
    <script src="https://muazkhan.com:9001/demos/menu.js"></script>
  </head>
  <body>
    <header>
      <a class="logo" href="/"
        ><img
          src="https://muazkhan.com:9001/demos/logo.png"
          alt="RTCMultiConnection"
      /></a>
      <a href="/" class="menu-explorer"
        >Menu<img
          src="https://muazkhan.com:9001/demos/menu-icon.png"
          alt="Menu"
      /></a>
      <nav>
        <li>
          <a href="/">Home</a>
        </li>
        <li>
          <a href="https://muazkhan.com:9001/demos/">Demos</a>
        </li>
        <li>
          <a href="https://www.rtcmulticonnection.org/docs/getting-started/"
            >Getting Started</a
          >
        </li>
        <li>
          <a href="https://www.rtcmulticonnection.org/FAQ/">FAQ</a>
        </li>
        <li>
          <a
            href="https://www.youtube.com/playlist?list=PLPRQUXAnRydKdyun-vjKPMrySoow2N4tl"
            >YouTube</a
          >
        </li>
        <li>
          <a href="https://github.com/muaz-khan/RTCMultiConnection/wiki"
            >Wiki</a
          >
        </li>
        <li>
          <a href="https://github.com/muaz-khan/RTCMultiConnection">Github</a>
        </li>
      </nav>
    </header>

    <h1>
      Video Conferencing using RTCMultiConnection
      <p class="no-mobile">
        Multi-user (many-to-many) video chat using mesh networking model.
      </p>
    </h1>

    <section class="make-center">
      <div>
        <label
          ><input type="checkbox" id="record-entire-conference" /> Record Entire
          Conference In The Browser?</label
        >
        <span id="recording-status" style="display: none;"></span>
        <button id="btn-stop-recording" style="display: none;">
          Stop Recording
        </button>
        <br /><br />

        <input
          type="text"
          id="room-id"
          value="abcdef"
          autocorrect="off"
          autocapitalize="off"
          size="20"
        />
        <button id="open-room">Open Room</button>
        <button id="join-room">Join Room</button>
        <button id="open-or-join-room">Auto Open Or Join Room</button>
      </div>

      <div id="videos-container" style="margin: 20px 0;"></div>

      <div
        id="room-urls"
        style="
          text-align: center;
          display: none;
          background: #f1eded;
          margin: 15px -10px;
          border: 1px solid rgb(189, 189, 189);
          border-left: 0;
          border-right: 0;
        "
      ></div>
    </section>

    <script src="https://muazkhan.com:9001/dist/RTCMultiConnection.min.js"></script>
    <script src="https://muazkhan.com:9001/node_modules/webrtc-adapter/out/adapter.js"></script>
    <script src="https://muazkhan.com:9001/socket.io/socket.io.js"></script>

    <!-- custom layout for HTML5 audio/video elements -->
    <link
      rel="stylesheet"
      href="https://muazkhan.com:9001/dev/getHTMLMediaElement.css"
    />
    <script src="https://muazkhan.com:9001/dev/getHTMLMediaElement.js"></script>

    <script src="https://muazkhan.com:9001/node_modules/recordrtc/RecordRTC.js"></script>
    <script>
      // ......................................................
      // .......................UI Code........................
      // ......................................................
      document.getElementById("open-room").onclick = function () {
        disableInputButtons();
        connection.open(document.getElementById("room-id").value, function (
          isRoomOpened,
          roomid,
          error
        ) {
          if (isRoomOpened === true) {
            showRoomURL(connection.sessionid);
          } else {
            disableInputButtons(true);
            if (error === "Room not available") {
              alert(
                "Someone already created this room. Please either join or create a separate room."
              );
              return;
            }
            alert(error);
          }
        });
      };

      document.getElementById("join-room").onclick = function () {
        disableInputButtons();
        connection.join(document.getElementById("room-id").value, function (
          isJoinedRoom,
          roomid,
          error
        ) {
          if (error) {
            disableInputButtons(true);
            if (error === "Room not available") {
              alert(
                "This room does not exist. Please either create it or wait for moderator to enter in the room."
              );
              return;
            }
            alert(error);
          }
        });
      };

      document.getElementById("open-or-join-room").onclick = function () {
        disableInputButtons();
        connection.openOrJoin(
          document.getElementById("room-id").value,
          function (isRoomExist, roomid, error) {
            if (error) {
              disableInputButtons(true);
              alert(error);
            } else if (connection.isInitiator === true) {
              // if room doesn't exist, it means that current user will create the room
              showRoomURL(roomid);
            }
          }
        );
      };

      // ......................................................
      // ..................RTCMultiConnection Code.............
      // ......................................................

      var connection = new RTCMultiConnection();

      // by default, socket.io server is assumed to be deployed on your own URL
      connection.socketURL = "https://muazkhan.com:9001/";

      // comment-out below line if you do not have your own socket.io server
      // connection.socketURL = 'https://rtcmulticonnection.herokuapp.com:443/';

      connection.socketMessageEvent = "video-conference-demo";

      connection.session = {
        audio: true,
        video: true
      };

      connection.sdpConstraints.mandatory = {
        OfferToReceiveAudio: true,
        OfferToReceiveVideo: true
      };

      // STAR_FIX_VIDEO_AUTO_PAUSE_ISSUES
      // via: https://github.com/muaz-khan/RTCMultiConnection/issues/778#issuecomment-524853468
      var bitrates = 512;
      var resolutions = "Ultra-HD";
      var videoConstraints = {};

      if (resolutions == "HD") {
        videoConstraints = {
          width: {
            ideal: 1280
          },
          height: {
            ideal: 720
          },
          frameRate: 30
        };
      }

      if (resolutions == "Ultra-HD") {
        videoConstraints = {
          width: {
            ideal: 1920
          },
          height: {
            ideal: 1080
          },
          frameRate: 30
        };
      }

      connection.mediaConstraints = {
        video: videoConstraints,
        audio: true
      };

      var CodecsHandler = connection.CodecsHandler;

      connection.processSdp = function (sdp) {
        var codecs = "vp8";

        if (codecs.length) {
          sdp = CodecsHandler.preferCodec(sdp, codecs.toLowerCase());
        }

        if (resolutions == "HD") {
          sdp = CodecsHandler.setApplicationSpecificBandwidth(sdp, {
            audio: 128,
            video: bitrates,
            screen: bitrates
          });

          sdp = CodecsHandler.setVideoBitrates(sdp, {
            min: bitrates * 8 * 1024,
            max: bitrates * 8 * 1024
          });
        }

        if (resolutions == "Ultra-HD") {
          sdp = CodecsHandler.setApplicationSpecificBandwidth(sdp, {
            audio: 128,
            video: bitrates,
            screen: bitrates
          });

          sdp = CodecsHandler.setVideoBitrates(sdp, {
            min: bitrates * 8 * 1024,
            max: bitrates * 8 * 1024
          });
        }

        return sdp;
      };
      // END_FIX_VIDEO_AUTO_PAUSE_ISSUES

      // https://www.rtcmulticonnection.org/docs/iceServers/
      // use your own TURN-server here!
      connection.iceServers = [];
      /*
      connection.iceServers.push({
        urls: "stun:muazkhan.com:3478",
        credential: "muazkh",
        username: "hkzaum"
      });
      connection.iceServers.push({
        urls: "turns:muazkhan.com:5349",
        credential: "muazkh",
        username: "hkzaum"
      });
      connection.iceServers.push({
        urls: "turn:muazkhan.com:3478",
        credential: "muazkh",
        username: "hkzaum"
      });
      */
      /*
      connection.iceServers.push({
        urls: "stun:stun.l.google.com:19302"
      });
      connection.iceServers.push({
        urls: "stun:stun.bippassistant.com.br:3478"
      });*/
      connection.iceServers.push({
        urls: "turn:turn.bippassistant.com.br:3478",
        credential: "user",
        username: "user"
      });

      connection.videosContainer = document.getElementById("videos-container");
      connection.onstream = function (event) {
        var existing = document.getElementById(event.streamid);
        if (existing && existing.parentNode) {
          existing.parentNode.removeChild(existing);
        }

        event.mediaElement.removeAttribute("src");
        event.mediaElement.removeAttribute("srcObject");
        event.mediaElement.muted = true;
        event.mediaElement.volume = 0;

        var video = document.createElement("video");

        try {
          video.setAttributeNode(document.createAttribute("autoplay"));
          video.setAttributeNode(document.createAttribute("playsinline"));
        } catch (e) {
          video.setAttribute("autoplay", true);
          video.setAttribute("playsinline", true);
        }

        if (event.type === "local") {
          video.volume = 0;
          try {
            video.setAttributeNode(document.createAttribute("muted"));
          } catch (e) {
            video.setAttribute("muted", true);
          }
        }
        video.srcObject = event.stream;

        var width = parseInt(connection.videosContainer.clientWidth / 3) - 20;
        var mediaElement = getHTMLMediaElement(video, {
          title: event.userid,
          buttons: ["full-screen"],
          width: width,
          showOnMouseEnter: false
        });

        connection.videosContainer.appendChild(mediaElement);

        setTimeout(function () {
          mediaElement.media.play();
        }, 5000);

        mediaElement.id = event.streamid;

        // to keep room-id in cache
        localStorage.setItem(
          connection.socketMessageEvent,
          connection.sessionid
        );

        chkRecordConference.parentNode.style.display = "none";

        if (chkRecordConference.checked === true) {
          btnStopRecording.style.display = "inline-block";
          recordingStatus.style.display = "inline-block";

          var recorder = connection.recorder;
          if (!recorder) {
            recorder = RecordRTC([event.stream], {
              type: "video"
            });
            recorder.startRecording();
            connection.recorder = recorder;
          } else {
            recorder.getInternalRecorder().addStreams([event.stream]);
          }

          if (!connection.recorder.streams) {
            connection.recorder.streams = [];
          }

          connection.recorder.streams.push(event.stream);
          recordingStatus.innerHTML =
            "Recording " + connection.recorder.streams.length + " streams";
        }

        if (event.type === "local") {
          connection.socket.on("disconnect", function () {
            if (!connect
            }
          });
        }
      };

      var recordingStatus = document.getElementById("recording-status");
      var chkRecordConference = document.getElementById(
        "record-entire-conference"
      );
      var btnStopRecording = document.getElementById("btn-stop-recording");
      btnStopRecording.onclick = function () {
        var recorder = connection.recorder;
        if (!recorder) return alert("No recorder found.");
        recorder.stopRecording(function () {
          var blob = recorder.getBlob();
          invokeSaveAsDialog(blob);

          connection.recorder = null;
          btnStopRecording.style.display = "none";
          recordingStatus.style.display = "none";
          chkRecordConference.parentNode.style.display = "inline-block";
        });
      };

      connection.onstreamended = function (event) {
        var mediaElement = document.getElementById(event.streamid);
        if (mediaElement) {
          mediaElement.parentNode.removeChild(mediaElement);
        }
      };

      connection.onMediaError = function (e) {
        if (e.message === "Concurrent mic process limit.") {
          if (DetectRTC.audioInputDevices.length <= 1) {
            alert(
              "Please select external microphone. Check github issue number 483."
            );
            return;
          }

          var secondaryMic = DetectRTC.audioInputDevices[1].deviceId;
          connection


          style.css
          {
  "name": "video-conferencing-using-rtcmulticonnection",
  "version": "1.0.0",
  "description": "",
  "main": "index.html",
  "scripts": {
    "start": "parcel index.html --open",
    "build": "parcel build index.html"
  },
  "dependencies": {},
  "devDependencies": {},
  "keywords": []
}
