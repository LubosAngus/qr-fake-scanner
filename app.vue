<script setup lang="ts">
import { Html5Qrcode } from "html5-qrcode";

const showError = ref(false)
const showOverlay = ref(false)
const scannedValue = ref('')

function closeOverlay() {
  showOverlay.value = false

  startScanning()
}

function closeError() {
  showError.value = false

  startScanning()
}

function onScanSuccess(decodedText: any, decodedResult: any) {
  let savedTickets;

  try {
    savedTickets = JSON.parse(localStorage.getItem('tickets') || '[]')
  } catch (error) {
    savedTickets = []
  }

  if (savedTickets.includes(decodedText)) {
    new Audio('/erro.mp3').play()

    showError.value = true
    return
  }

  // alert(`Code matched = ${decodedText}`, decodedResult);
  new Audio('/tada.mp3').play()

  showOverlay.value = true
  scannedValue.value = decodedText

  savedTickets.push(decodedText)

  localStorage.setItem('tickets', JSON.stringify(savedTickets))
}

onMounted(() => {
  startScanning()
})

async function startScanning() {
  Html5Qrcode.getCameras().then(devices => {
    /**
     * devices would be an array of objects of type:
     * { id: "id", label: "label" }
     */
    if (!devices || !devices.length) {
      return
    }

    var cameraId
    try {
      cameraId = devices[1].id;
    } catch (error) {
      cameraId = devices[0].id;
    }

    const html5QrCode = new Html5Qrcode(/* element id */ "reader");
    html5QrCode.start(
      cameraId,
      {
        fps: 15,    // Optional, frame per seconds for qr code scanning
        qrbox: {
          width: 120,
          height: 120
        }  // Optional, if you want bounded box UI
      },
      (decodedText, decodedResult) => {
        // do something when code is read
        onScanSuccess(decodedText, decodedResult);

        html5QrCode.stop().then(ignore => {
          // QR Code scanning is stopped.
        }).catch(err => {
          // Stop failed, handle it.
        });
      },
      (errorMessage) => {});
  });
}
</script>

<template>
  <div>
    <span class="loader"></span>

    <div class="interface">
      <div id="reader" width="600px"></div>
    </div>

    <div class="overlay error" v-if="showError">
      <pre>Tota paska už bola naskenovaná!</pre>

      <button @click="closeError">haha</button>
    </div>

    <div class="overlay" v-if="showOverlay">
      <div class="success-checkmark">
        <div class="check-icon">
          <span class="icon-line line-tip"></span>
          <span class="icon-line line-long"></span>
          <div class="icon-circle"></div>
          <div class="icon-fix"></div>
        </div>
      </div>

      <pre>{{ scannedValue }}</pre>

      <button @click="closeOverlay">OK</button>
    </div>
  </div>
</template>

<style>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.loader {
  z-index: 1;
  position: fixed;
  top: calc(50% - 32px);
  left: calc(50% - 32px);
  width: 64px;
  height: 64px;
  border: 5px solid #FFF;
  border-left-color: transparent;
  border-bottom-color: transparent;
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
  animation: rotation .5s linear infinite;
}

    @keyframes rotation {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
    }

html,
body {
  padding: 0;
  margin: 0;
  background-color: black;
}

* {
  box-sizing: border-box;
}


.interface {
  position: fixed;
  width: 100%;
  height: 100dvh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 30px;
  padding: 30px;
  z-index: 2;

  #reader {
    width: 100%;
    flex: 0 0 auto;
  }

  button {
    width: 100%;
    height: 70px;
    background-color: black;
    color: white;
    border: none;
    font-size: 20px;
    font-weight: 700;
  }
}

.overlay {
  --color-green: #4CAF50;

  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100dvh;
  z-index: 10;
  opacity: 0;
  animation: fadeIn 0.25s ease forwards;
  background-color: var(--color-green);

  &.error {
    --color-green: #c81616;
  }

  display: flex;
  flex-direction: column;
  padding: 100px 30px;
  box-sizing: border-box;

  font-size: 30px;
  font-weight: 700;
  color: white;
  text-align: center;

  button {
    width: 100%;
    height: 70px;
    background-color: white;
    border: none;
    font-size: 20px;
    font-weight: 700;
    margin: auto 0 0;
  }
}
/**
 * Extracted from: SweetAlert
 * Modified by: Istiak Tridip
 */
 .success-checkmark {
    width: 80px;
    height: 115px;
    margin: 0 auto;
    transform: scale(3) translateX(-30px);
    transform-origin: top left;
    margin-bottom: 200px;

    .check-icon {
        width: 80px;
        height: 80px;
        position: relative;
        border-radius: 50%;
        box-sizing: content-box;
        border: 4px solid #FFFFFF;

        &::before {
            top: 3px;
            left: -2px;
            width: 30px;
            transform-origin: 100% 50%;
            border-radius: 100px 0 0 100px;
        }

        &::after {
            top: 0;
            left: 30px;
            width: 60px;
            transform-origin: 0 50%;
            border-radius: 0 100px 100px 0;
            animation: rotate-circle 4.25s ease-in;
        }

        &::before, &::after {
            content: '';
            height: 100px;
            position: absolute;
            background: var(--color-green);
            transform: rotate(-45deg);
        }

        .icon-line {
            height: 5px;
            background-color: #FFFFFF;
            display: block;
            border-radius: 2px;
            position: absolute;
            z-index: 10;

            &.line-tip {
                top: 46px;
                left: 14px;
                width: 25px;
                transform: rotate(45deg);
                animation: icon-line-tip 0.75s;
            }

            &.line-long {
                top: 38px;
                right: 8px;
                width: 47px;
                transform: rotate(-45deg);
                animation: icon-line-long 0.75s;
            }
        }

        .icon-circle {
            top: -4px;
            left: -4px;
            z-index: 10;
            width: 80px;
            height: 80px;
            border-radius: 50%;
            position: absolute;
            box-sizing: content-box;
            border: 4px solid rgba(255, 255, 255, .5);
        }

        .icon-fix {
            top: 8px;
            width: 5px;
            left: 26px;
            z-index: 1;
            height: 85px;
            position: absolute;
            transform: rotate(-45deg);
            background-color: var(--color-green);
        }
    }
}

@keyframes rotate-circle {
    0% {
        transform: rotate(-45deg);
    }
    5% {
        transform: rotate(-45deg);
    }
    12% {
        transform: rotate(-405deg);
    }
    100% {
        transform: rotate(-405deg);
    }
}

@keyframes icon-line-tip {
    0% {
        width: 0;
        left: 1px;
        top: 19px;
    }
    54% {
        width: 0;
        left: 1px;
        top: 19px;
    }
    70% {
        width: 50px;
        left: -8px;
        top: 37px;
    }
    84% {
        width: 17px;
        left: 21px;
        top: 48px;
    }
    100% {
        width: 25px;
        left: 14px;
        top: 45px;
    }
}

@keyframes icon-line-long {
    0% {
        width: 0;
        right: 46px;
        top: 54px;
    }
    65% {
        width: 0;
        right: 46px;
        top: 54px;
    }
    84% {
        width: 55px;
        right: 0px;
        top: 35px;
    }
    100% {
        width: 47px;
        right: 8px;
        top: 38px;
    }
}
</style>
