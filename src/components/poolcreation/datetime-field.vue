<template>
  <q-input
    outlined
    v-model="value.date"
    :label="label"
    :disable="pool.status !== 'draft' &&
                        this.$route.params.id != undefined"
    :readonly="pool.status !== 'draft' &&
                        this.$route.params.id != undefined"
    mask="####-##-## ##:##"
    fill-mask="#"
  >
    <template v-slot:prepend>
      <q-icon name="event" class="cursor-pointer">
        <q-popup-proxy transition-show="scale" transition-hide="scale">
          <q-date v-model="value.date" :mask="dateTimeMask" today-btn>
            <div class="row items-center justify-end">
              <q-btn v-close-popup label="Close" color="primary" flat />
            </div>
          </q-date>
        </q-popup-proxy>
      </q-icon>
    </template>
    <template v-slot:append>
      <q-icon name="access_time" class="cursor-pointer">
        <q-popup-proxy transition-show="scale" transition-hide="scale">
          <q-time v-model="value.date" :mask="dateTimeMask" format24h now-btn>
            <div class="row items-center justify-end">
              <q-btn v-close-popup label="Close" color="primary" flat />
            </div>
          </q-time>
        </q-popup-proxy>
      </q-icon>
    </template>
  </q-input>
</template>

<script>
export default {
  data() {
    return {
      dateTimeMask: "YYYY-MM-DD HH:mm"
    };
  },
  props: {
    value: {},
    label: {},
    pool: {}
  }
};
</script>
