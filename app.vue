<template>
  <div>
    <div v-for="(_, index) in otpConfigs" :key="index">
      <p>Current Method: {{ otpStates[index].current_method.method }}</p>
      <p>Resend Attempts: {{ otpStates[index].otp_resend_attempt }}</p>
      <button @click="sendOtp(index)">Send OTP for {{ otpStates[index].current_method.method }}</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

interface Method {
  method: string;
  max_attempt_resend: number;
}

interface OtpConfig {
  default_method: Method;
  other_method: Method[];
}

interface OtpState {
  current_method: Method;
  otp_resend_attempt: number;
  other_method_index: number;
}

const otpConfigs: OtpConfig[] = [
  {
    default_method: {
      method: 'email',
      max_attempt_resend: 5,
    },
    other_method: [
      { 
        method: 'whatsapp',
        max_attempt_resend: 2,
      }
    ],
  },
  {
    default_method: {
      method: 'whatsapp',
      max_attempt_resend: 2,
    },
    other_method: [
      {
        method: 'sms',
        max_attempt_resend: 2,
      },
      {
        method: 'line',
        max_attempt_resend: 2,
      },
    ],
  },
  {
    default_method: {
      method: 'line',
      max_attempt_resend: 2,
    },
    other_method: [
      {
        method: 'sms',
        max_attempt_resend: 2,
      },
      {
        method: 'email',
        max_attempt_resend: 2,
      },
    ],
  },
];

const otpStates = ref<OtpState[]>(otpConfigs.map(config => ({
  current_method: config.default_method,
  otp_resend_attempt: 0,
  other_method_index: 0,
})));

console.log('otpStates', otpStates)
function sendOtp(index: number) {
  const currentConfig = otpConfigs[index];
  const otpState = otpStates.value[index];

  if (otpState.otp_resend_attempt < otpState.current_method.max_attempt_resend) {
    otpState.otp_resend_attempt++;
  } else if (currentConfig.other_method.length > otpState.other_method_index) {
    otpState.other_method_index++;
    otpState.current_method = currentConfig.other_method[otpState.other_method_index - 1];
    otpState.otp_resend_attempt = 1;
  }
}
</script>
