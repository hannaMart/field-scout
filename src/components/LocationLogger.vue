<template>
  <div class="logger">
    <section class="controls">
      <button @click="getLocation" :disabled="loading">
        {{ loading ? "Pobieranie GPS..." : "Pobierz Lokalizację" }}
      </button>

      <div v-if="coords" class="coords-box">
        <p>Szerokość: {{ coords.latitude.toFixed(4) }}</p>
        <p>Długość: {{ coords.longitude.toFixed(4) }}</p>
      </div>

      <textarea
        v-model="note"
        placeholder="Opisz co widzisz w terenie..."
        rows="4"
      ></textarea>

      <button @click="saveNote" class="btn-save" :disabled="!note || !coords">
        Zapisz obserwację
      </button>
    </section>

    <section class="history">
      <h3>Twoje obserwacje:</h3>
      <ul>
        <li v-for="(item, index) in logs" :key="index">
          <strong>{{ item.time }}</strong
          ><br />
          <small>{{ item.lat }}, {{ item.lng }}</small>
          <p>{{ item.content }}</p>
        </li>
      </ul>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const coords = ref(null);
const loading = ref(false);
const note = ref("");

// const logs = ref(JSON.parse(localStorage.getItem('field_logs') || '[]'));
const logs = ref([]);

let db;

const request = indexedDB.open("fieldScoutDB", 1);

request.onupgradeneeded = () => {
  request.result.createObjectStore("logs", {
    keyPath: "id",
    autoIncrement: true,
  });
};

request.onsuccess = () => {
  db = request.result;

  const tx = db.transaction("logs", "readonly");

  tx.objectStore("logs").getAll().onsuccess = (e) => {
    logs.value = e.target.result.reverse();
  };
};

const getLocation = () => {
  loading.value = true;
  if (!navigator.geolocation) {
    alert("Geolokalizacja nie jest wspierana!");
    return;
  }

  navigator.geolocation.getCurrentPosition(
    (position) => {
      coords.value = position.coords;
      loading.value = false;
    },
    (err) => {
      alert("Błąd GPS: " + err.message);
      loading.value = false;
    },
  );
};
// const getLocation = () => {
//   loading.value = true;

//   if (!navigator.geolocation) {
//     alert("Geolokalizacja nie jest wspierana w tej przeglądarce.");
//     loading.value = false;
//     return;
//   }

//   if (!window.isSecureContext) {
//     alert(
//       "Geolokalizacja wymaga HTTPS albo localhost. Otwórz aplikację na komputerze przez http://localhost:4173",
//     );
//     loading.value = false;
//     return;
//   }

//   navigator.geolocation.getCurrentPosition(
//     (position) => {
//       coords.value = position.coords;
//       loading.value = false;
//     },
//     (err) => {
//       console.log(err);

//       if (err.code === 1) {
//         alert(
//           "Brak zgody na geolokalizację. Sprawdź uprawnienia przeglądarki.",
//         );
//       } else if (err.code === 2) {
//         alert("Nie można ustalić lokalizacji.");
//       } else if (err.code === 3) {
//         alert("Przekroczono czas pobierania lokalizacji.");
//       } else {
//         alert("Błąd GPS: " + err.message);
//       }

//       loading.value = false;
//     },
//     {
//       enableHighAccuracy: true,
//       timeout: 10000,
//       maximumAge: 0,
//     },
//   );
// };

const saveNote = () => {
  const newLog = {
    time: new Date().toLocaleTimeString(),
    lat: coords.value.latitude,
    lng: coords.value.longitude,
    content: note.value,
  };

  // Na razie używamy tylko LocalStorage (zadanie dla studenta: przejść na IndexedDB)
  // logs.value.unshift(newLog);
  // localStorage.setItem("field_logs", JSON.stringify(logs.value));
  const tx = db.transaction("logs", "readwrite");
  tx.objectStore("logs").add(newLog);

  logs.value.unshift(newLog);

  // Reset formularza
  note.value = "";
};
</script>

<style scoped>
.logger {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.controls {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  background: white;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}
button {
  padding: 12px;
  background: var(--p-color);
  border: none;
  color: white;
  border-radius: 4px;
  font-weight: bold;
  cursor: pointer;
}
button:disabled {
  background: #ccc;
}
textarea {
  width: 100%;
  box-sizing: border-box;
  padding: 10px;
  border: 1px solid #ddd;
}
.coords-box {
  font-family: monospace;
  font-size: 0.9rem;
  background: #eee;
  padding: 5px;
  text-align: center;
}
.history ul {
  list-style: none;
  padding: 0;
}
.history li {
  background: white;
  margin-bottom: 0.5rem;
  padding: 10px;
  border-left: 4px solid var(--p-color);
  font-size: 0.9rem;
}
</style>
