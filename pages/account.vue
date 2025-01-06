<template>
    <div class="account-page">
      <div class="account-details">
        <h2>Kontodaten</h2>
  
        <!-- Formulare für E-Mail und Passwort nebeneinander -->
        <div class="form-row">
          <form class="container" @submit.prevent="updateEmail">
            <h3>Ändere deine E-Mail</h3>
            <label for="email"><b>E-Mail</b></label>
            <input v-model="email" type="email" name="email" required />
            <p v-if="emailErrorMsg" class="error">{{ emailErrorMsg }}</p>
            <p v-if="emailSuccessMsg" class="success">{{ emailSuccessMsg }}</p>
            <button type="submit">E-Mail ändern</button>
          </form>
  
          <form class="container" @submit.prevent="updatePassword">
            <h3>Ändere dein Passwort</h3>
            <label for="password"><b>Neues Passwort</b></label>
            <input v-model="password" type="password" name="password" required />
            <label for="passwordConfirm"><b>Passwort bestätigen</b></label>
            <input v-model="passwordConfirm" type="password" name="passwordConfirm" required />
            <p v-if="passwordErrorMsg" class="error">{{ passwordErrorMsg }}</p>
            <p v-if="passwordSuccessMsg" class="success">{{ passwordSuccessMsg }}</p>
            <button type="submit">Passwort ändern</button>
          </form>
        </div>
  
      </div>
    </div>
  </template>
  
  <style scoped>
  .account-page {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 30px 15px;
    height: 100%;
    width: 100%;
    box-sizing: border-box;
  }
  
  .account-details {
    width: 100%;
    max-width: 1100px;
    margin: 0 auto;
  }
  
  h2, h3 {
    text-align: center;
    margin-bottom: 20px;
  }
  
  .account-info {
    display: flex;
    flex-direction: column;
    padding: 15px;
    margin-bottom: 30px;
    background-color: var(--background1);
    border-radius: 8px;
    border: 1px solid #ccc;
  }
  
  .account-info div {
    margin-bottom: 10px;
  }
  
  .form-row {
    display: flex;
    justify-content: space-between;
    gap: 30px;
  }
  
  .container {
    background-color: var(--background1);
    display: flex;
    flex-direction: column;
    padding: 1.5rem;
    width: 48%;
    border: 1px solid #ccc;
    border-radius: 10px;
    margin-bottom: 1rem;
  }
  
  input {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    border: 1px solid #333;
    background-color: #f4f4f4;
    color: #333;
    border-radius: 4px;
  }
  
  button {
    padding: 12px 20px;
    margin-top: 20px;
    background-color: var(--accent1);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  button:disabled {
    background-color: #ccc;
  }
  
  p {
    font-size: 14px;
    margin-top: 10px;
  }
  
  .error {
    color: red;
  }
  
  .success {
    color: green;
  }
  </style>
  
  <script setup>
  const client = useSupabaseClient();
  const user = useSupabaseUser();
  const router = useRouter();
  
  const email = ref(user?.email || "");
  const password = ref("");
  const passwordConfirm = ref("");
  
  // Nachrichten für Fehler und Erfolg
  const emailErrorMsg = ref("");
  const passwordErrorMsg = ref("");
  const emailSuccessMsg = ref("");
  const passwordSuccessMsg = ref("");
  
  // E-Mail ändern
  async function updateEmail() {
    try {
      if (email.value === user?.email) {
        emailErrorMsg.value = "Die neue E-Mail-Adresse darf nicht mit der alten übereinstimmen.";
        emailSuccessMsg.value = "";
        return;
      }
  
      const { error } = await client.auth.updateUser({
        email: email.value,
      });
  
      if (error) {
        throw new Error(error.message);
      }
  
      emailSuccessMsg.value = "Deine E-Mail wurde erfolgreich geändert!";
      emailErrorMsg.value = "";
    } catch (error) {
      emailErrorMsg.value = error.message || "Ein Fehler ist aufgetreten.";
      emailSuccessMsg.value = "";
    }
  }
  
  // Passwort ändern
  async function updatePassword() {
    try {
      if (password.value !== passwordConfirm.value) {
        passwordErrorMsg.value = "Die Passwörter stimmen nicht überein.";
        passwordSuccessMsg.value = "";
        return;
      }
  
      const { error } = await client.auth.updateUser({
        password: password.value,
      });
  
      if (error) {
        throw new Error(error.message);
      }
  
      passwordSuccessMsg.value = "Dein Passwort wurde erfolgreich geändert!";
      passwordErrorMsg.value = "";
    } catch (error) {
      passwordErrorMsg.value = error.message || "Ein Fehler ist aufgetreten.";
      passwordSuccessMsg.value = "";
    }
  }
  
  </script>
  