<script lang="ts">
  import { goto } from '$app/navigation';
  import { login } from '$lib/auth';

  let email    = $state('');
  let password = $state('');
  let error    = $state('');
  let loading  = $state(false);

  const canSubmit = $derived(email.includes('@') && password.length > 0 && !loading);

  async function submit() {
    error   = '';
    loading = true;

    try {
      await login({ email, password });
      await goto('/dashboard');
    } catch (err) {
      const msg = err instanceof Error ? err.message : '';
      // Give a friendlier message for credential errors
      error = msg.toLowerCase().includes('invalid') || msg.toLowerCase().includes('401')
        ? 'Email or password is incorrect. Please try again.'
        : msg || 'Unable to log in. Please try again.';
    } finally {
      loading = false;
    }
  }
</script>

<svelte:head>
  <title>Log in | DevCard</title>
  <meta name="description" content="Log in to your DevCard account to manage your developer profile links." />
</svelte:head>

<main class="auth-page">
  <section class="auth-panel glass">
    <a href="/" class="brand">⚡ DevCard</a>
    <h1>Welcome back</h1>
    <p class="lede">Log in to manage your links and QR code.</p>

    <form onsubmit={(e) => { e.preventDefault(); void submit(); }}>

      <label>
        <span>Email</span>
        <input
          id="login-email"
          bind:value={email}
          type="email"
          autocomplete="email"
          required
          placeholder="ada@example.com"
        />
      </label>

      <label>
        <span>Password</span>
        <input
          id="login-password"
          bind:value={password}
          type="password"
          autocomplete="current-password"
          required
          placeholder="Your password"
        />
      </label>

      {#if error}
        <p class="form-error" role="alert">⚠ {error}</p>
      {/if}

      <button class="btn-primary" type="submit" disabled={!canSubmit}>
        {loading ? 'Logging in…' : 'Log in'}
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
    padding: 2.25rem 2rem;
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
    margin-bottom: 1.75rem;
  }

  h1 {
    font-size: 2rem;
    margin-bottom: 0.5rem;
  }

  .lede {
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
    gap: 0.4rem;
    color: var(--text-secondary);
    font-size: 0.9rem;
    font-weight: 700;
  }

  input {
    width: 100%;
    border: 1.5px solid var(--border);
    border-radius: 10px;
    background: var(--bg-card);
    color: var(--text-primary);
    font: inherit;
    padding: 0.85rem 1rem;
    transition: border-color 0.2s;
  }

  input:focus {
    border-color: var(--primary);
    outline: 3px solid rgba(99, 102, 241, 0.18);
  }

  button:disabled {
    cursor: not-allowed;
    opacity: 0.55;
  }

  .form-error {
    border-radius: 10px;
    background: rgba(239, 68, 68, 0.1);
    border: 1px solid rgba(239, 68, 68, 0.25);
    color: #b91c1c;
    padding: 0.8rem 1rem;
    font-size: 0.9rem;
    line-height: 1.5;
  }

  .switch {
    margin-top: 1.5rem;
    text-align: center;
    color: var(--text-secondary);
    font-size: 0.9rem;
  }

  .switch a {
    color: var(--primary);
    font-weight: 800;
  }
</style>
