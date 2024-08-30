<script>
  import Notification from './Components/Notification.svelte';

  let notificationMessage = "";
  let notificationType = "info";

  let projects = JSON.parse(localStorage.getItem("projects")) || [];
  let newProjectName = "";
  let startTime = "";
  let endTime = "";
  let workType = "";
  let selectedProjectIndex = 0;
  let icons = {
    delete: '<svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" viewBox="0 0 24 24"><path fill="currentColor" d="M19 4h-3.5l-1-1h-5l-1 1H5v2h14M6 19a2 2 0 0 0 2 2h8a2 2 0 0 0 2-2V7H6z"/></svg>',
    plus: '<svg xmlns="http://www.w3.org/2000/svg" width="1em" height="1em" viewBox="0 0 24 24"><path fill="currentColor" d="M17 13h-4v4h-2v-4H7v-2h4V7h2v4h4m-5-9A10 10 0 0 0 2 12a10 10 0 0 0 10 10a10 10 0 0 0 10-10A10 10 0 0 0 12 2"/></svg>'
  };

  function triggerNotification() {
    notificationMessage = "Операция выполнена успешно!";
    notificationType = "success";

    // Очистка сообщения через некоторое время, чтобы предотвратить повторное отображение
    setTimeout(() => {
      notificationMessage = "";
    }, 3000);
  }

  function addProject() {
    if (newProjectName) {
      projects.push({ name: newProjectName, days: {} });
      newProjectName = "";
      saveProjects();
    }
  }

  function saveProjects() {
    projects = [...projects]
    localStorage.setItem("projects", JSON.stringify(projects));
    triggerNotification();
  }

  function addTime() {
    if (startTime && endTime && workType) {
      const start = new Date(`1970-01-01T${startTime}:00`);
      const end = new Date(`1970-01-01T${endTime}:00`);

      const diffMs = end - start;
      const diffHours = diffMs / 1000 / 60 / 60;

      let totalHours;
      if (diffHours >= 0) {
        totalHours = diffHours;
      } else {
        totalHours = 24 + diffHours; // Для случаев, когда работа продолжается через полночь
      }

      const today = new Date().toISOString().split("T")[0];

      if (!projects[selectedProjectIndex].days[today]) {
        projects[selectedProjectIndex].days[today] = [];
      }
      projects[selectedProjectIndex].days[today].push({
        hours: totalHours,
        type: workType,
      });
      saveProjects();
    }
  }

  function deleteProject(index) {
    projects.splice(index, 1);
    saveProjects();
  }

  function deleteEntry(projectIndex, day, entryIndex) {
    projects[projectIndex].days[day].splice(entryIndex, 1);
    if (projects[projectIndex].days[day].length === 0) {
      delete projects[projectIndex].days[day];
    }
    saveProjects();
  }
</script>

<main>
  <h1>Трекер рабочего времени</h1>

  <div class="add-project-container d-flex">
    <label for="new-project" class="m-0">Добавить проект:</label>
    <input
      type="text"
      id="new-project"
      bind:value={newProjectName}
      placeholder="Название проекта"
      class="m-0"
    />
    <button on:click={addProject}>{@html icons.plus}</button>
  </div>

  <hr />

  {#if projects.length > 0}
    <div>
      <label for="project-select">Выберите проект:</label>
      <select id="project-select" bind:value={selectedProjectIndex}>
        {#each projects as project, index}
          <option value={index}>{project.name}</option>
        {/each}
      </select>
    </div>

    <div class="d-flex" style="gap: 10px;">
      <div class="w-50">
        <label for="start-time">Время начала:</label>
        <input type="time" id="start-time" bind:value={startTime} />
      </div>

      <div class="w-50">
        <label for="end-time">Время окончания:</label>
        <input type="time" id="end-time" bind:value={endTime} />
      </div>
    </div>

    <div>
      <label for="work-type">Тип работы:</label>
      <input
        type="text"
        id="work-type"
        bind:value={workType}
        placeholder="Тип работы"
      />
    </div>

    <div class="d-flex" style="justify-content: end;">
    <button on:click={addTime} class="m-0 my-1">Добавить трек времени</button>
    </div>

    <h2 class="py-2">Проекты</h2>
    <ul>
      {#each projects as project, projectIndex}
        <li>
          <div class="project-item w-100 py-2">
            <strong>{project.name}</strong>
            <button
              on:click={() => deleteProject(projectIndex)}
              class="btn delete">{@html icons.delete}</button
            >
          </div>
          <ul class="items w-100">
            {#each Object.entries(project.days) as [day, entries]}
              <li class="item">
                {day}
                <ul>
                  {#each entries as entry, entryIndex}
                    <li class="item-entry d-flex">
                    <span>{(entry.hours * 60).toFixed(0)} минут - {entry.type}</span>
                      <button on:click={() => deleteEntry(projectIndex, day, entryIndex)}
                        class="delete-entry-button"
                        >{@html icons.delete}</button
                      >
                    </li>
                  {/each}
                  <li style="margin-top: 1.5rem;">
                    <strong>Итог:</strong>
                    {entries.reduce((total, entry) => total + entry.hours, 0).toFixed(2)} часов
                    ({entries.reduce((total, entry) => total + entry.hours * 60, 0).toFixed(0)} минут)
                  </li>
                </ul>
              </li>
            {/each}
          </ul>
        </li>
      {/each}
    </ul>
  {/if}

  <Notification message={notificationMessage} type={notificationType} duration={3000} />
</main>

<style>
  hr {
      border: none;
      border-top: 1px solid silver;
      margin: 30px 0;
  }

  main {
    font-family: Arial, sans-serif;
    max-width: 500px;
    margin: 50px auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #f9f9f9;
  }

  label {
    display: block;
    margin-top: 10px;
  }

  input,
  select {
    width: 100%;
    padding: 5px;
    margin-top: 5px;
  }

  button {
    padding: 10px;
    margin-left: 5px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  button:hover {
    background-color: #0056b3;
  }

  ul {
    list-style-type: none;
    padding-left: 0;
  }

  li.item {
    margin-top: 10px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #e9e9e9;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  li button {
    display: flex;
    background-color: #dc3545;
    padding: 5px 10px;
    border-radius: 5px;
    margin: 0;
  }

  li button:hover {
    background-color: #c82333;
  }

  ul > li > div {
    display: flex;
    justify-content: space-between;
    width: 100%;
  }

  .w-100 {
    width: 100% !important;
  }
  .w-50 {
    width: 50% !important;
  }

  .d-flex {
    display: flex;
  }

  .py-2 {
    padding-top: 10px;
    padding-bottom: 10px;
  }

  .m-0 {
    margin: 0;
  }

  .my-1 {
    margin-bottom: .75rem;
    margin-top: .75rem;
  }

  .my-auto {
    margin-top: auto;
    margin-bottom: auto;
  }

  .my-3 {
    margin-bottom: 1.25rem;
    margin-top: 1.25rem;
  }

  .project-item {
    display: flex;
    align-items: center;
  }
  .project-item strong {
    text-transform: uppercase;
  }
  .project-item button.delete {
    margin-right: 11px;
    /* background-color: #ff6e7c; */
  }

  .items {
    margin-top: 1rem;
    padding-bottom: 1rem;
    border-bottom: 1px dotted silver;
  }
  .item ul {
    width: 65%;
  }
  .item button {
    background-color: unset;
    color: #dc3545;
  }
  .item button:hover {
    color: white;
  }

  .item-entry {
    justify-content: space-between;
  }

  .add-project-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .add-project-container button {
    border-radius: 50%;
    width: 38px;
    height: 28px;
    padding: 0;
    display: flex;
    align-items: center;
    margin: 0;
        margin-left: 0px;
    justify-content: center;
    margin-left: 10px;
    background-color: #1fb500;
  }
  .add-project-container button:hover {
    background-color: #1a9c00;
  }
</style>
