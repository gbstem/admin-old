<script>
  import { classNames } from '$lib/utils'
  import { doc, getDocs, collection, updateDoc } from 'firebase/firestore'
  import { db } from '$lib/firebase'
  import { serialize } from '$lib/forms'
  import { alert } from '$lib/stores'
  import { onMount } from 'svelte'
  import Papa from 'papaparse'

  const allApplications = []
  const allRegistrations = []
  let ready = false

  async function onUpload(event){
    const file = event.target.files[0]
    const text = await file.text()
    const parsed = Papa.parse(text, {header: true})
    const classes = parsed.data
    console.log(classes)
  }

  function formatRegistration(registration){
    
    const {personal, academic, program, agreements, meta} = registration

    const formattedRegistration = {
        email: personal.email.value,
        secondaryEmail: personal.secondaryEmail.value,
        studentFirstName: personal.studentFirstName.value,
        studentLastName: personal.studentLastName.value,
        parentFirstName: personal.parentFirstName.value,
        parentLastName: personal.parentLastName.value,
        phoneNumber: personal.phoneNumber.value,
        dateOfBirth: personal.dateOfBirth.value,
        gender: personal.gender.value,
        raceEthnicity: personal.raceEthnicity.value,
        frlp: personal.frlp.value,
        parentEducation: personal.parentEducation.value,
        address: personal.address.value,
        school: academic.school.value,
        grade: academic.grade.value,
        reason: program.reason.value,
        csCourse: program.csCourse.value,
        mathCouse: program.mathCourse.value,
        engineeringCourse: program.engineeringCourse.value,
        scienceCourse: program.scienceCourse.value,
        timeSlots: program.timeSlots.value.join(";"),
        inPerson: program.inPerson.value,
        id: meta.id.value,
        uid: meta.uid.value,
    }
    return formattedRegistration
  }


  function formatApplication(application){

      const {personal, academic, program, essay, meta} = application
  
      const formattedApplication = {
          email: personal.email.value,
          firstName: personal.firstName.value,
          lastName: personal.lastName.value,
          phoneNumber: personal.phoneNumber.value,
          dateOfBirth: personal.dateOfBirth.value,
          gender: personal.gender.value,
          raceEthnicity: personal.raceEthnicity.value,
          address: personal.address.value,
          school: academic.school.value,
          graduationYear: academic.graduationYear.value,
          reason: program.reason.value,
          preferences: program.preferences.value,
          numClasses: program.numClasses.value,
          courses: program.courses.value.join(";"),
          timeSlots: program.timeSlots.value.join(";"),
          inPerson: program.inPerson.value,
          academicBackground: essay.academicBackground.value,
          teachingScenario: essay.teachingScenario.value,
          why: essay.why.value,
          id: meta.id.value,
          uid: meta.uid.value,
      }
      return formattedApplication
      
    }
  

  onMount(async () => {
      const registrationSnapshot = await getDocs(collection($db, 'registrations'))
      registrationSnapshot.forEach(doc => {
        const registration = serialize.fromServer(doc.data())
        if (registration.meta.submitted.checked){
          allRegistrations.push(formatRegistration(registration))
        }
        if (registration.meta.uid.value !== doc.id) {
          alert.trigger(
            'error',
            'Registration UID mismatch. Please contact Yuen Ler before proceeding.',
            false
          )
        }
      })

      const applicationSnapshot = await getDocs(collection($db, 'applications'))
      applicationSnapshot.forEach(doc => {
        const application = serialize.fromServer(doc.data())
        if (application.meta.submitted.checked){
          allApplications.push(formatApplication(application))
        }
        if (application.meta.uid.value !== doc.id) {
          alert.trigger(
            'error',
            'Application UID mismatch. Please contact Yuen Ler before proceeding.',
            false
          )
        }
      })

      ready = true
    })
</script>

{#if ready}
<div class = "flex flex-col items-center justify-center">
<!-- 3 buttons: Download Registrations, download applications, upload classes -->
<div class="mb-5">  
<button 
  class = "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
  on:click={() => {
    const csv = Papa.unparse(allRegistrations)
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.setAttribute('href', url)
    link.setAttribute('download', 'registrations.csv')
    link.click()
  }}>Download Registrations</button>
</div>
  <div class="mb-5">
  <button 
  class = "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
  
  on:click={() => {
    const csv = Papa.unparse(allApplications)
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
    const url = URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.setAttribute('href', url)
    link.setAttribute('download', 'applications.csv')
    link.click()
  }}>Download Applications</button>
  </div>
  <div class="mb-5">
    <label 
    class="font-bold"
    >Upload Classes</label>
    <br>
  <input
  type="file" 
  accept=".csv"
  class = "bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
  on:change={async (event) => {
    onUpload(event)
  }}
  />
  </div>
</div>
{/if}