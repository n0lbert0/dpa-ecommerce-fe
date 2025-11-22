<template>
  <q-card class="q-pa-md" style="max-width: 700px; margin: 0 auto;">
    <q-card-section>
      <div class="text-h6">Registro</div>
    </q-card-section>

    <q-card-section>
      <q-form ref="registerForm" @submit.prevent="handleSubmit">
        <div class="row q-col-gutter-md">
          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.firstName" label="Nombre (FirstName)" :rules="[v => !!v || 'Requerido']" />
          </div>

          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.lastName" label="Apellido (LastName)" />
          </div>

          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.dateOfBirth" type="date" label="Fecha de nacimiento" />
          </div>

          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.country" label="País (Country)" />
          </div>

          <div class="col-12">
            <q-input outlined dense v-model="form.address" label="Dirección (Address)" />
          </div>

          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.email" type="email" label="Email" :rules="emailRules" />
          </div>

          <div class="col-12 col-md-6">
            <q-input outlined dense v-model="form.password" type="password" label="Contraseña" :rules="passwordRules" />
          </div>

        </div>

        <div class="row q-mt-md">
          <div class="col">
            <q-btn label="Registrar" type="submit" color="primary" />
            <q-btn label="Limpiar" flat class="q-ml-sm" color="secondary" @click="resetForm" />
          </div>
        </div>
      </q-form>
    </q-card-section>
  </q-card>
</template>

<script setup>
import { ref } from 'vue'
import { useQuasar } from 'quasar'
import { api } from 'boot/axios'

const $q = useQuasar()

const registerForm = ref(null)

const form = ref({
  firstName: '',
  lastName: '',
  dateOfBirth: null,
  country: '',
  address: '',
  email: '',
  password: '',
  type: null,
})


const emailRules = [
  v => !!v || 'Requerido',
  v => /\S+@\S+\.\S+/.test(v) || 'Email inválido'
]

const passwordRules = [
  v => !!v || 'Requerido',
  v => (v && v.length >= 6) || 'Mínimo 6 caracteres'
]

function formatDateToIso(dateString) {
  if (!dateString) return null
  // dateString comes as YYYY-MM-DD from input[type=date]
  return dateString
}

async function handleSubmit() {
  // validate form (Quasar QForm validate if available)
  let valid = true
  try {
    if (registerForm.value && registerForm.value.validate) {
      valid = await registerForm.value.validate()
    }
  } catch (e) {
    // If validate throws, fall back to true
    console.log('Validation error:', e);
    
  }

  if (!valid) {
    $q.notify({ type: 'negative', message: 'Corrige los errores del formulario.' })
    return
  }

  const payload = {
    FirstName: form.value.firstName,
    LastName: form.value.lastName || null,
    DateOfBirth: formatDateToIso(form.value.dateOfBirth),
    Country: form.value.country || null,
    Address: form.value.address || null,
    Email: form.value.email,
    Password: form.value.password,
    
  }

  try {
    const resp = await api.post('api/user/signup', payload)
    $q.notify({ type: 'positive', message: 'Registro exitoso.' })
    // opcional: limpiar form

    console.log('Respuesta del servidor:', resp.data);

    resetForm()
  } catch (err) {
    const message = err?.response?.data?.message || err.message || 'Error en el registro.'
    $q.notify({ type: 'negative', message })
  }
}

function resetForm() {
  form.value = {
    firstName: '',
    lastName: '',
    dateOfBirth: null,
    country: '',
    address: '',
    email: '',
    password: '',
    type: null,
  }
  if (registerForm.value && registerForm.value.resetValidation) {
    registerForm.value.resetValidation()
  }
}
</script>

<style scoped>
.q-card { box-shadow: var(--q-card-box-shadow); }
</style>