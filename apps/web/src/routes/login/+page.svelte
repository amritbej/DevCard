<script lang="ts">
  import { goto } from '$app/navigation';
  import { login } from '$lib/auth';

  let email = $state('');
  let password = $state('');
  let error = $state('');
  let loading = $state(false);

  async function submit() {
    error = '';
    loading = true;

    try {
      await login({ email, password });
      await goto('/dashboard');
    } catch (err) {
      error = err instanceof Error ? err.message : 'Unable to log in.';
    } finally {
      loading = false;
    }
  }
</script>

<svelte:head>
  <title>Log in | DevCard</title>
</svelte:head>

<main class="auth-page">
  <section class="auth-panel glass">
    <a href="/" class="brand">DevCard</a>
    <h1>Log in</h1>
    <p class="lede">Open your dashboard and keep your profile links current.</p>

    <form onsubmit={(event) => { event.preventDefault(); void submit(); }}>
      <label>
        <span>Email</span>
        <input bind:value={email} type="email" autocomplete="email" required />
      </label>

      <label>
        <span>Password</span>
        <input bind:value={password} type="password" autocomplete="current-password" required />
      </label>

      {#if error}
        <p class="form-error" role="alert">{error}</p>
      {/if}

      <button class="btn-primary" type="submit" disabled={loading}>
        {loading ? 'Logging in...' : 'Log in'}
      </button>
    </form>

    <p class="switch">New to DevCard? <a href="/signup">Create an account</a></p>
  </section>
</main>

<style>
  .auth-page {
    min-height: 100vh;
    display: grid;
    place-items: center;
    padding: 2rem 1rem;
  }

  .auth-panel {
    width: min(100%, 440px);
    border-radius: var(--radius);
    padding: 2rem;
    background: rgba(255, 255, 255, 0.78);
  }

  :global(html.dark) .auth-panel {
    background: rgba(15, 23, 42, 0.82);
  }

  .brand {
    display: inline-flex;
    font-family: 'Outfit', sans-serif;
    font-size: 1.1rem;
    font-weight: 800;
    margin-bottom: 2rem;
  }

  h1 {
    font-size: 2.2rem;
    margin-bottom: 0.75rem;
  }

  .lede,
  .switch {
    color: var(--text-secondary);
    line-height: 1.6;
  }

  form {
    display: grid;
    gap: 1rem;
    margin-top: 1.75rem;
  }

  label {
    display: grid;
    gap: 0.45rem;
    color: var(--text-secondary);
    font-weight: 700;
  }

  input {
    width: 100%;
    border: 1px solid var(--border);
    border-radius: 10px;
    background: var(--bg-card);
    color: var(--text-primary);
    font: inherit;
    padding: 0.9rem 1rem;
  }

  input:focus {
    border-color: var(--primary);
    outline: 3px solid rgba(99, 102, 241, 0.18);
  }

  button:disabled {
    cursor: wait;
    opacity: 0.75;
  }

  .form-error {
    border-radius: 10px;
    background: rgba(239, 68, 68, 0.12);
    color: #b91c1c;
    padding: 0.8rem 1rem;
  }

  .switch {
    margin-top: 1.5rem;
    text-align: center;
  }

  .switch a {
    color: var(--primary);
    font-weight: 800;
  }
</style>
