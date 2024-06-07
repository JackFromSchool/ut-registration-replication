<script lang="ts">
   import classNames from "$lib/classNames";
   import courseTypes from "$lib/courseTypes";
   import meetingInfos from "$lib/meetingInfos";
      
   enum Status {
      None,
      Closed,
      Success,
      SuccessDropAdd,
      SuccessDrop,
      ClosedNoWaitlist,
   }

   class Course {
      unique: string;
      course: string;
      meeting_info: string;
      title: string;

      constructor(unique: string, course: string, meeting_info: string, title: string) {
         this.unique = unique;
         this.course = course;
         this.meeting_info = meeting_info;
         this.title = title;
      }
   }
   
   let status: Status = Status.None;
   let last_unique_number: string = "";
   
   let choice: string = "";
   let unique_number: string = "";
   let dependent_upon_number: string = "no_drop";
   let dependent_upon_replace: string = "";
   let drop_number: string = "no_drop";

   let reset_fields = () => {
      choice = "";
      unique_number = "";
      dependent_upon_number = "no_drop";
      dependent_upon_replace = "";
      drop_number = "no_drop";
   }
   
   let courses: Course[] = [];

   let handle_submit = () => {
      if(choice === "add") {
         if(unique_number.length !== 5) {
            return;
         }

         if(Math.floor(Math.random()*3) == 0) {
            let course = courseTypes[Math.floor(Math.random()*courseTypes.length)];
            let meeting_info = meetingInfos[Math.floor(Math.random()*meetingInfos.length)];
            let title = classNames[Math.floor(Math.random()*classNames.length)];
            courses.push(new Course(unique_number, course, meeting_info, title));
            courses = courses;
            status = Status.Success;
            reset_fields();
         } else {
            last_unique_number = unique_number;
            status = Status.ClosedNoWaitlist;
            reset_fields();
         }
         
      } else if(choice === "drop") {
         if(drop_number === "no_drop") {
            return;
         }
         
         courses = courses.filter(obj => obj.unique != drop_number);
         status = Status.SuccessDrop;
         reset_fields();
      } else if(choice === "dropdependenton") {
         if(dependent_upon_number === "no_drop" || dependent_upon_replace.length !== 5) {
            return;
         }

         if(Math.floor(Math.random()*3) == 0) {
            let course = courseTypes[Math.floor(Math.random()*courseTypes.length)];
            let meeting_info = meetingInfos[Math.floor(Math.random()*meetingInfos.length)];
            let title = classNames[Math.floor(Math.random()*classNames.length)];
            courses.push(new Course(dependent_upon_replace, course, meeting_info, title));
            courses = courses;

            courses = courses.filter(obj => obj.unique != dependent_upon_number);
            
            status = Status.SuccessDropAdd;
            reset_fields();
         } else {
            last_unique_number = dependent_upon_replace;
            status = Status.ClosedNoWaitlist;
            reset_fields();
         }
      } 
   };
   
</script>

<h1>
   Registration
</h1>
{#if status == Status.Closed}
<p class="failure">Add was unsuccessful because:</p>
<p class="failure">The class requested is closed.</p>
{:else if status == Status.ClosedNoWaitlist}
<p class="failure">Add was unsuccessful because:</p>
<p class="failure">The class requested is closed. There is no waitlist for this class.</p>
{:else if status == Status.Success}
<p class="success">Class successfully added.</p>
{:else if status == Status.SuccessDrop}
<p class="success">Class successfully dropped.</p>
{:else if status == Status.SuccessDropAdd}
<p class="success">Class successfully added and requested other class dropped.</p>
{/if}
<table>
   <tr>
      <th>SELECT</th>
      <th>UNIQUE NUMBER</th>
      <th>TRANSACTION</th>
   </tr>
   {#if status == Status.ClosedNoWaitlist}
   <tr>
      <td><input type="radio" name="choice" value="search" bind:group={choice}></td>
      <td>{last_unique_number}</td>
      <td>SEARCH for another section of the same course.</td>
   </tr>
   {/if}
   <tr>
      <td><input type="radio" name="choice" value="add" bind:group={choice}></td>
      <td><input type="text" maxlength="5" bind:value={unique_number}></td>
      <td>REGISTER for or ADD</td>
   </tr>
   {#if courses.length != 0}
   <tr>
      <td><input type="radio" name="choice" value="drop" bind:group={choice}></td>
      <td>
         <select bind:value={drop_number}>
            <option value="no_drop">
               Select unique to:
            </option>
            {#each courses as course}
               <option value={course.unique}>
                  {course.unique}
               </option>
            {/each}
         </select>
      </td>
      <td>DROP</td>
   </tr>
   <tr>
     <td><input type="radio" name="choice" value="dropdependenton" bind:group={choice}></td>
     <td>
         <select bind:value={dependent_upon_number}>
            <option value="no_drop">
               Select unique to:
            </option>
            {#each courses as course}
               <option value={course.unique}>
                  {course.unique}
               </option>
            {/each}
         </select>
      </td>
      <td>
         DROP DEPENDENT UPON successfully ADDING <input type="text" maxlength="5" bind:value={dependent_upon_replace}>
      </td>
   </tr>
   {/if}
</table>
<button on:click={handle_submit}>Submit</button>
<table>
   <tr>
      <th>Unique</th>
      <th>Course</th>
      <th>Meeting Information</th>
      <th>Title</th>
   </tr>
   {#each courses as course}
   <tr>
      <td>{course.unique}</td>
      <td>{course.course}</td>
      <td>{course.meeting_info}</td>
      <td>{course.title}</td>
   </tr>
   {/each}
</table>

<style>

   h1, p, button {
      margin-left: 1rem;
   }

   button {
      margin-top: 3rem;
      margin-bottom: 4rem;
   }
   
   table {
      table-layout: fixed;
      border-spacing: 1rem;
   }

   th {
      text-align: left;
   }

   input[type="text"] {
      width: 3em;
   }

   .failure {
      color: red;
   }

   .success {
      color: green;
   }

</style>
