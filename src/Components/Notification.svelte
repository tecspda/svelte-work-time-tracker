<script>
  export let message = "";
  export let type = "info"; // Тип уведомления: 'info', 'success', 'error' и т.д.
  export let duration = 3000; // Длительность отображения в миллисекундах

  let visible = false;

  // Показываем уведомление на короткое время
  function show() {
    visible = true;
    setTimeout(() => {
      visible = false;
    }, duration);
  }

  $: if (message) {
    show();
  }
</script>

<style>
  .notification {
    position: fixed;
    top: 0;
    right: 0;
    margin: 1rem;
    padding: 1rem 2rem;
    border-radius: 8px;
    color: white;
    background-color: #333;
    opacity: 0;
    transform: translateY(-100%);
    transition: transform 0.5s ease, opacity 0.5s ease;
    z-index: 1000;
  }

  .notification.show {
    opacity: 1;
    transform: translateY(0);
  }

  .info {
    background-color: #3498db;
  }

  .success {
    background-color: #2ecc71;
  }

  .error {
    background-color: #e74c3c;
  }
</style>

<div class="notification {type} {visible ? 'show' : ''}">
  {message}
</div>
