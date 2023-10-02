<script setup>
import { ref, onMounted, computed } from 'vue';
import { useVuelidate } from '@vuelidate/core'
import { required, helpers, minLength, maxLength, minValue, maxValue } from '@vuelidate/validators'
import Table from './Table.vue';
import stateCity from '../assets/stateCities.json'
const formData = ref({
  FirstName: '', LastName: '', DateOfBirth: '', Gender: '', Age: '', Hobbies: [], State: '', City: ''
})

const rules = computed(() => {
  return {
    FirstName: {
      required: helpers.withMessage("First Name Can't be Blank *", required),
      minLength: helpers.withMessage("First Name must contain atleast 3 characters !", minLength(3)),
      maxLength: helpers.withMessage("First Name must contain atmost 20 characters !", maxLength(20)),
      containsOnlyCharacters: helpers.withMessage('Only Characters are allowed!', (val) => /^[A-Za-z]+$/.test(val))
    },
    LastName: {
      required: helpers.withMessage("Last Name Can't be Blank *", required),
      minLength: helpers.withMessage("Last Name must contain atleast 3 characters !", minLength(3)),
      maxLength: helpers.withMessage("Last Name must contain atmost 20 characters !", maxLength(20)),
      containsOnlyCharacters: helpers.withMessage('Only Characters are allowed!', (val) => /^[A-Za-z]+$/.test(val))
    },
    DateOfBirth: {
      required: helpers.withMessage("Please Select Date of Birth*", required)
    },
    Gender: {
      required: helpers.withMessage("Please select Gender*", required)
    },
    Age: {
      required: helpers.withMessage("Age Required *", required),
      minValue: helpers.withMessage("Not Eligible", minValue(18)),
      maxValue: helpers.withMessage("Not Eligible", maxValue(100)),
    },
    Hobbies: {
      required: helpers.withMessage("Please select hobby*", required)
    },
    State: {
      required: helpers.withMessage("Please Select a State*", required),
    },
    City: {
      required: helpers.withMessage("Please Select a City*", required),
    }
  }
})

const v$ = useVuelidate(rules, formData.value)
const errors = ref({
  sameDataError: ''
})
const allUsers = ref([])
const allStates = ref([])
const allCities = ref([])
const edit = ref({ isEdit: false, editIndex: -1 })

onMounted(
  () => {
    allStates.value = Object.keys(stateCity).sort()
    allUsers.value = JSON.parse(localStorage.getItem('voting_userdata'))
  }
)
const handleStateChange = (e) => {
  formData.value.City = ''
  allCities.value = stateCity[e].sort()
}

const setLocalStorageData = (data) => {
  localStorage.setItem('voting_userdata', JSON.stringify(data))
}

const resetForm = () => {
  formData.value.FirstName = "",
    formData.value.LastName = "",
    formData.value.Age = "",
    formData.value.Gender = ""
  formData.value.Hobbies = []
  formData.value.State = ""
  formData.value.City = ""
  formData.value.DateOfBirth = ""
}
async function handleSubmit() {
  let res = await v$.value.$validate()
  if (res) {
    errors.value.sameDataError = ''
    if (edit.value.editIndex >= 0) {
      let restUser = allUsers.value.filter(user => user !== allUsers.value[edit.value.editIndex])
      if (restUser.some((user) => user.FirstName.trim() === formData.value.FirstName.trim() && user.LastName.trim() === formData.value.LastName.trim() && user.Age === formData.value.Age && user.Gender === formData.value.Gender && JSON.stringify(formData.value.Hobbies.sort()) === JSON.stringify(user.Hobbies.sort()) && user.DateOfBirth === formData.value.DateOfBirth && user.State === formData.value.State && user.City === formData.value.City)) {
        errors.value.sameDataError = "Record already exist!"
      }
      else {
        allUsers.value[edit.value.editIndex] = formData.value
        setLocalStorageData(allUsers.value)
        edit.value.editIndex = -1
        edit.value.isEdit = false
        resetForm()
        v$.value.$reset()
      }
    }
    else {
      if (allUsers.value?.some((user) => user.FirstName.trim() === formData.value.FirstName.trim() && user.LastName.trim() === formData.value.LastName.trim() && user.Age === formData.value.Age && user.Gender === formData.value.Gender && JSON.stringify(formData.value.Hobbies.sort()) === JSON.stringify(user.Hobbies.sort()) && user.DateOfBirth === formData.value.DateOfBirth && user.State === formData.value.State && user.City === formData.value.City)) {
        errors.value.sameDataError = "Record already exist!"
      }
      else {
        if (localStorage.getItem('voting_userdata')) {
          setLocalStorageData([formData.value, ...JSON.parse(localStorage.getItem('voting_userdata'))])
        }
        else {
          setLocalStorageData([formData.value])
        }
        resetForm()
        v$.value.$reset()
      }
    }
    allUsers.value = JSON.parse(localStorage.getItem('voting_userdata'))
  }
}

const handleDelete = (index) => {
  allUsers.value = allUsers.value.filter((user) => allUsers.value.indexOf(user) !== index)
  setLocalStorageData(allUsers.value)
}

const handleEdit = (index) => {
  edit.value.isEdit = true
  edit.value.editIndex = index
  const editItem = allUsers.value[index]

  formData.value.FirstName = editItem.FirstName
  formData.value.LastName = editItem.LastName
  formData.value.DateOfBirth = editItem.DateOfBirth
  formData.value.Gender = editItem.Gender
  formData.value.Age = editItem.Age
  formData.value.Hobbies = editItem.Hobbies
  formData.value.State = editItem.State
  handleStateChange(editItem.State)
  formData.value.City = editItem.City
}

onMounted(() => {
  allUsers.value = JSON.parse(localStorage.getItem('voting_userdata'))
})
</script>
<template>
  <div class="container my-2 m-auto">
    <div class="row py-3">
      <h2 class="mx-2">Voting Registration Form</h2>
      <div class="col-12">
        <form @submit.prevent="handleSubmit" class="p-3 rounded border">
          <span class="formError text-danger" id="SameDataError">{{ errors.sameDataError }}</span>
          <div class="form-group mb-3">
            <label for="firstName">First Name</label>
            <input class="form-control" type="text" name="firstName" id="FirstName" v-model.trim="formData.FirstName" />
            <span class="formError text-danger" id="firstNameError">{{ v$.FirstName.$errors[0]?.$message }}</span>
          </div>
          <div class="form-group mb-3">
            <label for="lastName">Last Name</label>
            <input class="form-control" type="text" id="lastName" name="LastName" v-model.trim="formData.LastName" />
            <span class="formError text-danger" id="lastNameError">{{ v$.LastName.$errors[0]?.$message }}</span>
          </div>
          <div class="row justify-content-between">
            <div class="form-group mb-2 col-sm-6">
              <label for="dateOfBirth">Date Of Birth</label>
              <input id="dateOfBirth" name="DOB" class="form-control" type="date" v-model="formData.DateOfBirth" />
              <span class="formError text-danger" id="dateOfBirthError">{{ v$.DateOfBirth.$errors[0]?.$message }}</span>
            </div>
            <div class="form-group mb-2 col-sm-6">
              <label for="age">Age</label>
              <input id="age" class="form-control" type="number" v-model="formData.Age" />
              <span class="formError text-danger" id="ageError">{{ v$.Age.$errors[0]?.$message }}</span>
            </div>
          </div>
          <div class="row justify-content-between">
            <div class="form-group mb-3 col-6">
              <label for="Hobbies">Hobbies</label>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Chess"
                  id="Chess">
                <label class="form-check-label" for="Chess">
                  Chess
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies"
                  value="Badminton" id="Badminton">
                <label class="form-check-label" for="Badminton">
                  Badminton
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Hockey"
                  id="Hockey">
                <label class="form-check-label" for="Hockey">
                  Hockey
                </label>
              </div>
              <div class="form-check">
                <input class="form-check-input" name="Hobbies" type="checkbox" v-model="formData.Hobbies" value="Reading"
                  id="Reading">
                <label class="form-check-label" for="Reading">
                  Reading
                </label>
              </div>
              <span class="formError text-danger" id="HobbiesError">{{ v$.Hobbies.$errors[0]?.$message }}</span>
            </div>
            <div class="form-group mb-2 col-6">
              <label for="Gender">Gender</label>
              <div class="form-check">
                <input class="form-check-input" id="male" name="Gender" value="Male" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="male">Male</label>
              </div>
              <div class="form-check">
                <input class="form-check-input" id="female" name="Gender" value="Female" type="radio"
                  v-model="formData.Gender" /><label class="form-check-label" for="female">Female</label>
              </div>
              <span class="formError text-danger" id="GenderError">{{ v$.Gender.$errors[0]?.$message }}</span>
            </div>
          </div>

          <div class="row justify-content-between">
            <div class="form-group mb-3 col-sm-6">
              <label for="stateDropdown">Select a State</label>
              <select id="stateDropdown" class="form-select" v-model="formData.State"
                @change="handleStateChange($event.target.value)">
                <option value="" disabled selected>Select a State</option>
                <option :value="state" v-for="state in allStates">{{ state }}</option>
              </select>
              <span class="formError text-danger" id="stateError">{{ v$.State.$errors[0]?.$message }}</span>
            </div>
            <div class="form-group mb-3 col-sm-6">
              <label for="cityDropdown">Select a City</label>
              <select id="cityDropdown" class="form-select" v-model="formData.City">
                <option value="" selected disabled>Select a City</option>
                <option :value="city" v-for="city in allCities">{{ city }}</option>
              </select>
              <span class="formError text-danger" id="cityError">{{ v$.City.$errors[0]?.$message }}</span>
            </div>
          </div>

          <button :id="edit.isEdit ? 'update' : 'submit'" class="btn btn-success text-light">{{ !edit.isEdit ? 'Add' :
            'Update' }}</button>
        </form>
      </div>
    </div>
  </div>
  <Table :data="allUsers" :handleEdit="handleEdit" :handleDelete="handleDelete" />
</template>