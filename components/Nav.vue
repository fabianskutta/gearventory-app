<template>
    <nav class="nav-container">
      <img class="logo" src="/Gearventory.png" alt="">
      <ul class="nav-list">
        <li>
          <NuxtLink to="/" class="nav-link" exact-active-class="active">
            <span class="icon">🏠</span>
            Dashboard
          </NuxtLink>
        </li>
        <li>
          <NuxtLink to="/items" class="nav-link" exact-active-class="active">
            <span class="icon">📦</span>
            Materialien
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="nav-link" exact-active-class="active">
            <span class="icon">📋</span>
            Projekte
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="nav-link" exact-active-class="active">
            <span class="icon">👤</span>
            Kontakte
          </NuxtLink>
        </li>
        <li>
          <NuxtLink class="nav-link" exact-active-class="active">
            <span class="icon">📷</span>
            Scanner
          </NuxtLink>
        </li>


      </ul>
      <div class="account-section">
        <NuxtLink to="/account" class="nav-link account-link" exact-active-class="active">
          <span class="icon">👤</span>
          Dein Konto
        </NuxtLink>
        <button class="logout-button" @click="signOut()">
          <span class="icon">🚪</span>
          Logout
        </button>
      </div>
    </nav>
  </template>
  
  <script setup lang="ts">
    const client = useSupabaseClient();
    const user = useSupabaseUser()
    const router = useRouter();

  async function signOut() {
    const { error } = await client.auth.signOut();
    router.push("/login");
}
  </script>
  
  <style>
.logo {
  margin-bottom: 1rem;
}

  .nav-container {
    width: 250px;
    background-color: var(--background2);
    padding: 20px;
    box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
    height: 100vh;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    font-weight: 500;
  }
  
  .nav-list {
    list-style: none;
    margin: 0;
    padding: 0;
    flex-grow: 1;
    overflow-y: auto;
  }
  
  .nav-container li {
    margin-bottom: 20px;
  }
  
  .nav-link,
  .logout-button {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    text-decoration: none;
    color: var(--text2);
    font-size: 1rem;
    padding: 1rem;
    width: 100%;
    box-sizing: border-box;
    transition: background-color 0.3s ease;
    border-radius: 20px;
    background: none; /* Für Logout-Button */
    border: none; /* Für Logout-Button */
    cursor: pointer;
  }
  
  .nav-link .icon,
  .logout-button .icon {
    margin-right: 10px;
    font-size: 1.2rem;
  }
  
  .nav-link:hover,
  .nav-link.active,
  .logout-button:hover {
    background-color: #ffffff49;
    color: var(--text1);
  }
  
  .account-section {
    margin-top: auto;
    padding-top: 20px;
    border-top: 1px solid var(--divider); /* Optional: Trennungslinie */
  }
  
  .logout-button {
    margin-top: 10px;
    text-align: left; /* Optional: Positionierung des Textes */
  }
  </style>
  