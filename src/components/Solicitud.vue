<template>
  <div>
    <header class="header">
      <img :src="logo" alt="Logo Descubre tu barrio" class="logo" />
      <nav>
        <RouterLink to="/foro" class="btn btn-outline">
          <i class="fas fa-arrow-left"></i> Volver al foro
        </RouterLink>
      </nav>
    </header>

    <main class="form-container">
      <div class="form-header">
        <h1><i class="fas fa-seedling"></i> Solicitud de adopción de espacio público</h1>
        <p>
          Gracias por tu interés en cuidar tu comunidad. Completa este formulario para adoptar un
          parque, avenida o plaza y contribuir a su mantenimiento.
        </p>
      </div>

      <form class="adoption-form" @submit.prevent="handleSubmit">
        <!-- INFORMACIÓN PERSONAL -->
        <section class="form-section">
          <h2><i class="fas fa-user"></i> Información personal</h2>
          <div class="form-row">
            <div class="form-group">
              <label for="fullname">Nombre completo *</label>
              <input type="text" id="fullname" v-model="formData.fullname" placeholder="Ej. Ana López" required />
            </div>
            <div class="form-group">
              <label for="email">Correo electrónico *</label>
              <input type="email" id="email" v-model="formData.email" placeholder="Ej. ana@email.com" required />
            </div>
          </div>

          <div class="form-row">
            <div class="form-group">
              <label for="phone">Teléfono *</label>
              <input type="tel" id="phone" v-model="formData.phone" placeholder="Ej. 0987654321" required />
            </div>
            <div class="form-group">
              <label for="organization">Organización (opcional)</label>
              <input type="text" id="organization" v-model="formData.organization" placeholder="Nombre de tu grupo o comunidad" />
            </div>
          </div>
        </section>

        <!-- ESPACIO A ADOPTAR -->
        <section class="form-section">
          <h2><i class="fas fa-map-marker-alt"></i> Espacio a adoptar</h2>
          <div class="form-group">
            <label for="space-name">Nombre del espacio *</label>
            <input type="text" id="space-name" v-model="formData.spaceName" placeholder="Ej. Parque Central Norte" required />
          </div>

          <div class="form-group">
            <label for="space-location">Ubicación exacta *</label>
            <input
              type="text"
              id="space-location"
              v-model="formData.spaceLocation"
              placeholder="Dirección o punto de referencia"
              required
            />
            <MapSelector
              v-model="selectedLocation"
              @locationSelected="onLocationSelected"
            />
            <small>Haz clic en el mapa para seleccionar la ubicación exacta del espacio.</small>
          </div>
        </section>

        <!-- PLAN DE MANTENIMIENTO -->
        <section class="form-section">
          <h2><i class="fas fa-leaf"></i> Plan de mantenimiento</h2>
          <div class="form-group">
            <label>¿Qué actividades realizarás? *</label>
            <div class="checkbox-group">
              <label><input type="checkbox" value="cleaning" v-model="formData.activities" /> Limpieza</label>
              <label><input type="checkbox" value="gardening" v-model="formData.activities" /> Jardinería</label>
              <label><input type="checkbox" value="painting" v-model="formData.activities" /> Pintura</label>
              <label><input type="checkbox" value="repairs" v-model="formData.activities" /> Reparaciones</label>
              <label><input type="checkbox" value="other" v-model="formData.activities" /> Otra</label>
            </div>
            <div v-if="formData.activities.includes('other')" class="other-activity-panel">
              <label for="otherActivity">Especifica otra actividad:</label>
              <input type="text" id="otherActivity" v-model="formData.otherActivity" placeholder="Describe la actividad..." />
            </div>
          </div>

          <div class="form-group">
            <label for="frequency">Frecuencia estimada *</label>
            <select id="frequency" v-model="formData.frequency" required>
              <option value="">Seleccione...</option>
              <option value="weekly">Semanal</option>
              <option value="biweekly">Quincenal</option>
              <option value="monthly">Mensual</option>
              <option value="quarterly">Trimestral</option>
              <option value="punctual">Evento único</option>
            </select>
          </div>

          <div class="form-group">
            <label for="volunteers">Número estimado de voluntarios *</label>
            <input type="number" id="volunteers" v-model="formData.volunteers" min="1" placeholder="Ej. 10" required />
          </div>

          <div class="form-group">
            <label for="description">Descripción del plan *</label>
            <textarea
              id="description"
              v-model="formData.description"
              rows="4"
              placeholder="Describe brevemente las acciones y metas de tu adopción"
              required
            ></textarea>
          </div>
        </section>

        <!-- DOCUMENTACIÓN -->
        <section class="form-section">
          <h2><i class="fas fa-file-upload"></i> Documentación</h2>
          <div class="form-group">
            <label for="id-document">Documento de identidad *</label>
            <input type="file" id="id-document" accept=".pdf,.jpg,.jpeg,.png" required />
          </div>

          <div class="form-group">
            <label for="proposal">Propuesta o plan detallado (opcional)</label>
            <input type="file" id="proposal" accept=".pdf,.doc,.docx" />
          </div>
        </section>

        <div class="form-actions">
          <button type="submit" class="btn btn-primary btn-lg">
            <i class="fas fa-paper-plane"></i> Enviar solicitud
          </button>
        </div>
      </form>
    </main>
  </div>
</template>

<script setup>
defineOptions({ name: 'SolicitudAdopcion' })
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import MapSelector from './MapSelector.vue'
import { addAdoption } from '@/stores/adoptionsStore'

const router = useRouter()
const logo = new URL('@/assets/logo.png', import.meta.url).href

// Form data
const formData = ref({
  fullname: '',
  email: '',
  phone: '',
  organization: '',
  spaceName: '',
  spaceLocation: '',
  activities: [],
  otherActivity: '',
  frequency: '',
  volunteers: '',
  description: ''
})

const selectedLocation = ref(null)

// Handle location selection
function onLocationSelected(location) {
  formData.value.spaceLocation = location.address
}

// Handle form submission
async function handleSubmit() {
  // Validate required fields
  if (!formData.value.fullname || !formData.value.email || !formData.value.phone ||
      !formData.value.spaceName || !formData.value.spaceLocation ||
      !formData.value.frequency || !formData.value.volunteers || !formData.value.description) {
    alert('Por favor, completa todos los campos requeridos.')
    return
  }

  try {
    // Add adoption to Supabase
    const result = await addAdoption(formData.value)

    if (result.success) {
      console.log('Nueva adopción creada:', result.adoption)

      // Show success message
      alert('¡Solicitud enviada exitosamente! Tu adopción ha sido registrada y está pendiente de aprobación.')

      // Reset form
      resetForm()

      // Redirect to adoptions view
      router.push('/adopciones')
    } else {
      alert('Error: ' + result.error)
    }

  } catch (error) {
    console.error('Error al enviar la solicitud:', error)
    alert('Hubo un error al enviar la solicitud. Por favor, intenta nuevamente.')
  }
}

// Reset form function
function resetForm() {
  formData.value = {
    fullname: '',
    email: '',
    phone: '',
    organization: '',
    spaceName: '',
    spaceLocation: '',
    activities: [],
    otherActivity: '',
    frequency: '',
    volunteers: '',
    description: ''
  }
  selectedLocation.value = null
}
</script>

<style scoped>
/* Panel para especificar otra actividad */
.other-activity-panel {
  margin-top: 1rem;
  padding: 1rem;
  background: var(--light-bg);
  border-radius: 8px;
  border: 1px solid var(--gray-border);
}
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css');

/* Variables de color */
:root {
  --primary-green: #2E8B57;
  --secondary-green: #98FB98;
  --light-bg: #F5F5F5;
  --dark-text: #333333;
  --white: #FFFFFF;
  --gray-border: #E0E0E0;
  --hover-green: #3CB371;
}

/* Estructura principal */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background-color: var(--white);
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.logo {
  height: 50px;
}

.btn {
  padding: 0.8rem 1.5rem;
  border-radius: 25px;
  font-weight: 600;
  border: none;
  cursor: pointer;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  text-decoration: none;
}

.btn-outline {
  background-color: transparent;
  border: 2px solid var(--primary-green);
  color: var(--primary-green);
}

.btn-outline:hover {
  background-color: var(--primary-green);
  color: var(--white);
}

/* Contenedor del formulario */
.form-container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.form-header {
  text-align: center;
  margin-bottom: 2rem;
}

.form-header h1 {
  color: var(--primary-green);
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
}

.form-header p {
  color: var(--dark-text);
}

/* Estilos del formulario */
.adoption-form {
  background-color: var(--white);
  border-radius: 10px;
  padding: 2rem;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}

.form-section {
  margin-bottom: 2.5rem;
  padding-bottom: 2rem;
  border-bottom: 1px solid var(--gray-border);
}

.form-section:last-child {
  border-bottom: none;
}

.form-section h2 {
  color: var(--primary-green);
  font-size: 1.2rem;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  gap: 10px;
}

.form-row {
  display: flex;
  gap: 1.5rem;
  margin-bottom: 1rem;
}

.form-row .form-group {
  flex: 1;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-group label {
  display: flex;
  align-items: center;
  margin-bottom: 0.5rem;
  font-weight: 600;
  gap: 8px;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 0.8rem;
  border: 1px solid var(--gray-border);
  border-radius: 5px;
  font-size: 1rem;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
  outline: none;
  border-color: var(--primary-green);
  box-shadow: 0 0 0 2px rgba(46, 139, 87, 0.2);
}

.form-group textarea {
  resize: vertical;
  min-height: 100px;
}

.checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.checkbox-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: normal;
  cursor: pointer;
}

.map-placeholder {
  height: 200px;
  background-color: var(--light-bg);
  margin-top: 10px;
  border-radius: 5px;
  border: 1px dashed var(--gray-border);
}

.form-actions {
  text-align: center;
  margin-top: 2rem;
}

.btn-primary {
  background-color: var(--primary-green);
  color: var(--white);
}

.btn-primary:hover {
  background-color: var(--hover-green);
  transform: translateY(-2px);
}

.btn-lg {
  padding: 1rem 2rem;
  font-size: 1.1rem;
}

/* Estilos para los inputs de archivo */
input[type="file"] {
  padding: 0.5rem;
  border: 1px dashed var(--gray-border);
  background-color: var(--light-bg);
}

/* Iconos */
i.fas {
  color: var(--primary-green);
  width: 40px;
  text-align: center;
}

/* Responsive */
@media (max-width: 768px) {
  .form-row {
    flex-direction: column;
    gap: 0;
  }

  .form-container {
    padding: 0;
  }

  .adoption-form {
    padding: 1.5rem;
  }

  .header {
    flex-direction: column;
    gap: 1rem;
    padding: 1rem;
  }

  .logo {
    margin-bottom: 0.5rem;
  }

  .checkbox-group {
    flex-direction: column;
    gap: 0.5rem;
  }
}
</style>
