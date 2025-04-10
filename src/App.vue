<template>
  <div class="min-h-screen bg-gray-100 flex flex-col">
    <!-- Header -->
    <header class="bg-white shadow-sm">
      <div class="container mx-auto px-4 py-4">
        <h1 class="text-xl font-semibold text-gray-800">Voice Recorder</h1>
      </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 container mx-auto px-4 py-6">
      <div class="max-w-md mx-auto bg-white rounded-lg shadow-md p-6">
        <!-- Recording Status -->
        <div class="text-center mb-6">
          <div v-if="!isRecording && !audioUrl" class="text-lg font-medium">
            Ready to record
          </div>
          <div v-else-if="isRecording" class="text-lg font-medium text-red-500">
            Recording... {{ formatTime(recordingDuration) }}
          </div>
          <div v-else-if="audioUrl && !isSending" class="text-lg font-medium text-green-500">
            Recording complete
          </div>
          <div v-else-if="isSending" class="text-lg font-medium text-blue-500">
            Sending to Telegram...
          </div>
        </div>

        <!-- Audio Player -->
        <div v-if="audioUrl && !isSending" class="mb-6">
          <audio ref="audioPlayer" :src="audioUrl" controls class="w-full"></audio>
        </div>

        <!-- Recording Visualization -->
        <div class="flex justify-center items-center h-24 mb-6 bg-gray-50 rounded-lg audio-recorder-container">
          <!-- Updated UI with new animations -->
          <button
            @click="toggleRecording"
            :disabled="isSending"
            :class="{
              recording: isRecording,
              sending: isSending,
            }"
            class="record-button"
          >
            <div class="icon-container">
              <!-- Microphone icon with animation -->
              <svg
                v-if="!isRecording && !audioUrl"
                class="mic-icon"
                xmlns="http://www.w3.org/2000/svg"
                width="18"
                height="18"
                viewBox="0 0 24 24"
              >
                <g
                  fill="none"
                  stroke="currentColor"
                  stroke-linejoin="round"
                  stroke-width="1.5"
                >
                  <rect width="7" height="13.5" x="8.5" y="2" rx="3.5" />
                  <path
                    stroke-linecap="round"
                    d="M4.5 11.5c0 4.142 3.358 7.5 7.5 7.5s7.5-3.358 7.5-7.5M12 19v3"
                  />
                </g>
              </svg>
              
              <!-- Sound waves animation when recording -->
              <svg
                v-if="isRecording"
                class="recording-icon"
                xmlns="http://www.w3.org/2000/svg"
                width="18"
                height="18"
                viewBox="0 0 24 24"
              >
                <defs>
                  <mask id="ipTVoiceOne0">
                    <g fill="none" stroke="#fff" stroke-width="1.5">
                      <path
                        fill="#555555"
                        d="M12 22c5.523 0 10-4.477 10-10S17.523 2 12 2 2 6.477 2 12s4.477 10 10 10Z"
                      />
                      <path
                        stroke-linecap="round"
                        d="M15 9v6m3-4v2m-9-4v6m-3-4v2m6-6v10"
                      />
                    </g>
                  </mask>
                </defs>
                <path
                  fill="currentColor"
                  d="M0 0h24v24H0z"
                  mask="url(#ipTVoiceOne0)"
                />
              </svg>
              
              <!-- Refresh Icon -->
              <svg 
                v-if="audioUrl && !isRecording && !isSending"
                class="mic-icon"
                xmlns="http://www.w3.org/2000/svg"
                width="18"
                height="18"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="1.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <path d="M3 12a9 9 0 1 0 9-9 9.75 9.75 0 0 0-6.74 2.74L3 8"></path>
                <path d="M3 3v5h5"></path>
              </svg>
              
              <!-- Sending animation -->
              <svg
                v-if="isSending"
                class="sending-icon"
                xmlns="http://www.w3.org/2000/svg"
                width="18"
                height="18"
                viewBox="0 0 24 24"
              >
                <g
                  fill="none"
                  stroke="currentColor"
                  stroke-linejoin="round"
                  stroke-width="1.5"
                >
                  <path d="M15.5 12V5.5a3.5 3.5 0 1 0-7 0V12a3.5 3.5 0 1 0 7 0Z" />
                  <path
                    stroke-linecap="round"
                    d="M4.5 11.5c0 4.142 3.358 7.5 7.5 7.5c.877 0 1.718-.15 2.5-.427M19.5 11.5a7.475 7.475 0 0 1-.624 3M12 19v3m9-1L3 3"
                  />
                </g>
              </svg>
            </div>
          </button>
        </div>

        <!-- Controls -->
        <div class="flex justify-center space-x-4">
          <button 
            v-if="audioUrl && !isSending" 
            @click="sendToTelegram" 
            class="flex items-center justify-center w-16 h-16 rounded-full bg-blue-500 focus:outline-none">
            <!-- Telegram Icon -->
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="m22 2-7 20-4-9-9-4Z"></path>
              <path d="M22 2 11 13"></path>
            </svg>
          </button>
        </div>
      </div>
    </main>

    <!-- Toast Notification -->
    <transition name="fade">
      <div 
        v-if="showToast" 
        class="fixed bottom-4 right-4 px-4 py-3 rounded-lg shadow-lg"
        :class="{
          'bg-red-500 text-white': isToastError,
          'bg-green-500 text-white': !isToastError
        }">
        <div class="flex items-center">
          <span>{{ toastMessage }}</span>
          <button @click="closeToast" class="ml-3 text-white">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <path d="M18 6 6 18"></path>
              <path d="m6 6 12 12"></path>
            </svg>
          </button>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import axios from 'axios';

export default {
  name: 'VoiceRecorder',
  setup() {
    const isRecording = ref(false);
    const audioUrl = ref(null);
    const mediaRecorder = ref(null);
    const audioChunks = ref([]);
    const isSending = ref(false);
    const audioBlob = ref(null);
    const telegramBotToken = ref('');
    const telegramChatId = ref('');
    const showToast = ref(false);
    const toastMessage = ref('');
    const isToastError = ref(false);
    const recordingDuration = ref(0);
    const recordingTimer = ref(null);
    const audioPlayer = ref(null);

    const fetchBotConfig = async () => {
      try {
        telegramBotToken.value = '7831190114:AAHzGzHJ-eWctnYrRDrvpB9-vFdX52tuE-o';
        telegramChatId.value = '1176934921';
      } catch (error) {
        console.error(error);
        showErrorToast('Failed to load config');
      }
    };

    const showErrorToast = (msg) => {
      toastMessage.value = msg;
      isToastError.value = true;
      showToast.value = true;
      setTimeout(closeToast, 5000);
    };

    const showSuccessToast = (msg) => {
      toastMessage.value = msg;
      isToastError.value = false;
      showToast.value = true;
      setTimeout(closeToast, 5000);
    };

    const closeToast = () => {
      showToast.value = false;
    };

    const startRecording = async () => {
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

        mediaRecorder.value = new MediaRecorder(stream);
        audioChunks.value = [];

        mediaRecorder.value.ondataavailable = (e) => audioChunks.value.push(e.data);

        mediaRecorder.value.onstop = () => {
          const blob = new Blob(audioChunks.value, { type: 'audio/webm' });
          audioBlob.value = blob;
          audioUrl.value = URL.createObjectURL(blob);
          stream.getTracks().forEach(track => track.stop());
          clearInterval(recordingTimer.value);
          recordingTimer.value = null;
        };

        mediaRecorder.value.start();
        isRecording.value = true;
        recordingDuration.value = 0;

        recordingTimer.value = setInterval(() => {
          recordingDuration.value += 1;
        }, 1000);
      } catch (err) {
        console.error(err);
        showErrorToast('Mic access denied or error occurred.');
      }
    };

    const stopRecording = () => {
      if (mediaRecorder.value && isRecording.value) {
        mediaRecorder.value.stop();
        isRecording.value = false;
      }
    };

    const resetRecording = () => {
      if (audioUrl.value) URL.revokeObjectURL(audioUrl.value);
      audioUrl.value = null;
      audioBlob.value = null;
      audioChunks.value = [];
      recordingDuration.value = 0;
      if (audioPlayer.value) {
        audioPlayer.value.pause();
        audioPlayer.value.currentTime = 0;
      }
    };

    const toggleRecording = () => {
      if (!isRecording.value && !audioUrl.value) {
        startRecording();
      } else if (isRecording.value) {
        stopRecording();
      } else if (audioUrl.value) {
        resetRecording();
        startRecording();
      }
    };

    const sendToTelegram = async () => {
      if (!audioBlob.value || !telegramBotToken.value || !telegramChatId.value) {
        showErrorToast('Missing config or recording');
        return;
      }

      isSending.value = true;
      try {
        const formData = new FormData();
        formData.append('chat_id', telegramChatId.value);
        formData.append('audio', audioBlob.value, 'recording.webm');

        const res = await axios.post(
          `https://api.telegram.org/bot${telegramBotToken.value}/sendAudio`,
          formData,
          { headers: { 'Content-Type': 'multipart/form-data' } }
        );

        if (res.data?.ok) {
          showSuccessToast('Sent successfully!');
          resetRecording();
        } else {
          throw new Error('Telegram error');
        }
      } catch (err) {
        console.error(err);
        showErrorToast('Failed to send to Telegram');
      } finally {
        isSending.value = false;
      }
    };

    const formatTime = (s) => {
      const m = Math.floor(s / 60);
      const r = s % 60;
      return `${m.toString().padStart(2, '0')}:${r.toString().padStart(2, '0')}`;
    };

    onMounted(() => {
      fetchBotConfig();
    });

    return {
      isRecording,
      audioUrl,
      isSending,
      showToast,
      toastMessage,
      isToastError,
      recordingDuration,
      audioPlayer,
      toggleRecording,
      sendToTelegram,
      closeToast,
      formatTime
    };
  }
};
</script>

<style>
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.animate-pulse {
  animation: pulse 1s ease-in-out infinite;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* New UI and animation styles */
.audio-recorder-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.record-button {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  border: none;
  background: #4caf50;
  color: white;
  cursor: pointer;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  transition: all 0.3s ease;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

.record-button:hover {
  transform: scale(1.05);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
}

.record-button.recording {
  background: #f44336;
  animation: recordingPulse 1.5s infinite;
}

.record-button.sending {
  background: #2196f3;
  animation: none;
}

.record-button:disabled {
  background: #9e9e9e;
  cursor: not-allowed;
}

.icon-container {
  width: 18px;
  height: 18px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.mic-icon {
  width: 18px;
  height: 18px;
  transition: all 0.3s ease;
}

.recording-icon {
  width: 18px;
  height: 18px;
  animation: pulseIcon 1.5s infinite;
}

.sending-icon {
  width: 18px;
  height: 18px;
  animation: spin 1.5s linear infinite;
}

@keyframes recordingPulse {
  0% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(244, 67, 54, 0.7);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 0 0 6px rgba(244, 67, 54, 0.4);
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(244, 67, 54, 0);
  }
}

@keyframes pulseIcon {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.8;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
