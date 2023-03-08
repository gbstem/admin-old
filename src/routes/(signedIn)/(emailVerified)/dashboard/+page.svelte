<script>
  import Card from '$lib/components/Card.svelte'
  import { getDoc, doc } from 'firebase/firestore'
  import { fade } from 'svelte/transition'
  import { user, db } from '$lib/firebase'

  let uid = ''

  let data = {}
  let spreadsheet = []

  const getClasses = classes => {
    let data = {}
    const classIds = []
    for (const row of classes) {
      if (row.instructorUid === uid || row.instructor2Uid === uid || row.studentUid === uid) {
        classIds.push(row.classId)
      }
    }
    for (const row of classes) {
      if (classIds.includes(row.classId)) {
        if (data[row.classId]) {
          data[row.classId] = {
            ...data[row.classId],
            students: [
              ...data[row.classId].students,
              row.studentFirstName + ' ' + row.studentLastName
            ]
          }
        } else {
          data[row.classId] = {
            course: row.Course,
            instructorName: row.instructorName,
            instructor2Name: row.instructor2Name,
            location:
              row.Loc === 'CPL Main Branch0' || row.Loc === 'CPL Main Branch'
                ? 'CPL Main Branch'
                : 'Virtual',
            students: [row.studentFirstName + ' ' + row.studentLastName]
          }
        }
      }
    }
    return data
  }

  const asyncData = new Promise((resolve, reject) => {
    let data = {
      course: '',
      instructors: [],
      students: []
    }
    user.loaded().then(() => {
      // get classes doc
      getDoc(doc($db, 'classes', 'classes')).then(doc => {
        if (doc.exists()) {
          spreadsheet = doc.data().classes
        }
      })
    })
  })
</script>

<!-- {#await asyncData then data} -->
<input
  type="text"
  bind:value={uid}
  style="border : 1px solid black"
  on:change={() => {
    data = getClasses(spreadsheet)
  }}
/>

<div class="grid grid-cols-2" transition:fade|local={{ duration: 150 }}>
  {#if data && Object.keys(data).length > 0}
    {#each Object.entries(data) as [key, singleClass]}
      <Card>
        <div class="text-lg font-bold">{singleClass.course}</div>
        <div class="text-sm text-gray-500">Location</div>
        <div>{singleClass.location}</div>
        <div class="text-sm text-gray-500">Instructor(s)</div>
        <ul class="list-disc list-inside">
          <li>{singleClass.instructorName}</li>
          {#if singleClass.instructor2Name !== ''}
            <li>{singleClass.instructor2Name}</li>
          {/if}
        </ul>
        <div class="text-sm text-gray-500">Students</div>
        <ul class="list-disc list-inside">
          {#each singleClass.students as student}
            <li>{student}</li>
          {/each}
        </ul>
      </Card>
    {/each}
  {:else}
    <Card>
      <svelte:fragment slot="title">Class</svelte:fragment>
      <div class="text-sm text-gray-500">
        It seems like you're not teaching/enrolled in any class.
      </div>
    </Card>
  {/if}
</div>
<!-- {:catch} -->
<!-- <div>Error loading data. Please try again.</div> -->
<!-- {/await} -->
