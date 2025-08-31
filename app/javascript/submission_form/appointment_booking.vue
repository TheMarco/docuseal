<template>
  <div
    id="appointment_booking"
    class="mx-auto max-w-md flex flex-col appointment-booking-form"
    dir="auto"
  >
    <div class="font-medium text-2xl flex items-center space-x-1.5 mx-auto mb-4">
      <IconCircleCheck
        class="inline text-green-600"
        :width="30"
        :height="30"
      />
      <span class="appointment-booking-title">
        {{ t('document_signed_book_appointment') }}
      </span>
    </div>
    
    <div class="mb-4 text-center text-gray-600">
      <p>{{ t('great_next_step_schedule_meeting') }}</p>
    </div>

    <!-- Calendly Inline Widget -->
    <div
      id="calendly-inline-widget"
      class="calendly-inline-widget"
      style="min-width:320px;height:630px;"
    />

    <!-- Fallback buttons if Calendly fails to load -->
    <div
      v-if="showFallbackButtons"
      class="space-y-3 mt-4"
    >
      <a
        v-if="calendlyUrl"
        :href="calendlyUrl"
        target="_blank"
        rel="noopener noreferrer"
        class="white-button flex items-center w-full justify-center"
      >
        <IconCalendar class="mr-2" />
        <span>{{ t('schedule_appointment') }}</span>
      </a>
      
      <button
        class="white-button flex items-center w-full justify-center"
        @click="skipAppointment"
      >
        <span>{{ t('skip_for_now') }}</span>
      </button>
    </div>

    <!-- Skip button (always visible) -->
    <div class="mt-4 text-center">
      <button
        class="text-sm text-gray-500 hover:text-gray-700 underline"
        @click="skipAppointment"
      >
        {{ t('skip_appointment_booking') }}
      </button>
    </div>
  </div>
</template>

<script>
import { IconCircleCheck, IconCalendar } from '@tabler/icons-vue'

export default {
  name: 'AppointmentBooking',
  components: {
    IconCircleCheck,
    IconCalendar
  },
  inject: ['t'],
  props: {
    calendlyUrl: {
      type: String,
      required: false,
      default: 'https://calendly.com/your-username'
    },
    submitterName: {
      type: String,
      required: false,
      default: ''
    },
    submitterEmail: {
      type: String,
      required: false,
      default: ''
    }
  },
  data () {
    return {
      showFallbackButtons: false,
      calendlyLoaded: false
    }
  },
  mounted () {
    this.loadCalendly()
  },
  methods: {
    loadCalendly () {
      // Load Calendly widget script
      if (!window.Calendly) {
        const script = document.createElement('script')
        script.src = 'https://assets.calendly.com/assets/external/widget.js'
        script.onload = () => {
          this.initializeCalendly()
        }
        script.onerror = () => {
          console.error('Failed to load Calendly script')
          this.showFallbackButtons = true
        }
        document.head.appendChild(script)
      } else {
        this.initializeCalendly()
      }
    },
    
    initializeCalendly () {
      try {
        // Clear any existing widget
        const widget = document.getElementById('calendly-inline-widget')
        if (widget) {
          widget.innerHTML = ''
        }

        // Initialize Calendly inline widget
        window.Calendly.initInlineWidget({
          url: this.calendlyUrl,
          parentElement: document.getElementById('calendly-inline-widget'),
          prefill: {
            name: this.submitterName,
            email: this.submitterEmail
          },
          utm: {
            utmSource: 'docuseal',
            utmMedium: 'document_signing',
            utmCampaign: 'post_signature_booking'
          }
        })

        this.calendlyLoaded = true

        // Listen for Calendly events
        window.addEventListener('message', this.handleCalendlyEvent)
      } catch (error) {
        console.error('Failed to initialize Calendly widget:', error)
        this.showFallbackButtons = true
      }
    },

    handleCalendlyEvent (e) {
      if (e.data.event && e.data.event.indexOf('calendly') === 0) {
        if (e.data.event === 'calendly.event_scheduled') {
          // Appointment was successfully scheduled
          this.$emit('appointment-scheduled', e.data.event_details)
          this.showCompletionMessage()
        }
      }
    },

    showCompletionMessage () {
      // Show success message and then proceed to final completion
      setTimeout(() => {
        this.$emit('booking-complete')
      }, 2000)
    },

    skipAppointment () {
      this.$emit('booking-skipped')
    }
  },

  beforeUnmount () {
    window.removeEventListener('message', this.handleCalendlyEvent)
  }
}
</script>

<style scoped>
.appointment-booking-form {
  animation: fadeIn 0.5s ease-in-out;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.calendly-inline-widget {
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}
</style>
