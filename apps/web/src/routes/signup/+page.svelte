<script lang="ts">
  import { goto } from '$app/navigation';
  import { signup } from '$lib/auth';

  let displayName = $state('');
  let username    = $state('');
  let email       = $state('');
  let password    = $state('');
  let error       = $state('');
  let loading     = $state(false);

  // Per-field touched state so hints only show after the user interacts
  let touchedUsername = $state(false);
  let touchedPassword = $state(false);

  // Client-side validation
  const usernameOk  = $derived(/^[A-Za-z0-9_-]{3,50}$/.test(username));
  const passwordOk  = $derived(password.length >= 8);
  const passwordStr = $derived(
    password.length === 0 ? 0 :
    password.length < 6   ? 1 :
    password.length < 10  ? 2 : 3
  );
  const passwordStrLabel = $derived(['', 'Weak', 'Fair', 'Strong'][passwordStr]);
  const passwordStrColor = $derived(['', '#ef4444', '#f59e0b', '#22c55e'][passwordStr]);

  const canSubmit = $derived(
    displayName.trim().length > 0 &&
    usernameOk &&
    email.includes('@') &&
    passwordOk &&
    !loading
  );

  async function submit() {
    error   = '';
    loading = true;
    touchedUsername = true;
    touchedPassword = true;

    if (!canSubmit) {
      loading = false;
      if (!usernameOk)  error = 'Username must be 3–50 characters: letters, numbers, _ or - only.';
      else if (!passwordOk) error = 'Password must be at least 8 characters.';
      return;
    }

    try {
      await signup({ displayName: displayName.trim(), username: username.trim(), email, password });
      await goto('/dashboard');
    } catch (err) {
      error = err instanceof Error ? err.message : 'Unable to create your account.';
    } finally {
      loading = false;
    }
  }
</script>

<svelte:head>
  <title>Create account | DevCard</title>
  <meta name="description" content="Sign up for DevCard and share all your developer profiles through one QR code." />
</svelte:head>

<main class="auth-page">
  <section class="auth-panel glass">
    <a href="/" class="brand">⚡ DevCard</a>
    <h1>Create your DevCard</h1>
    <p class="lede">Add your links, get one QR, share everywhere.</p>

    <form onsubmit={(e) => { e.preventDefault(); void submit(); }}>

      <!-- Display Name -->
      <label>
        <span>Display name <span class="req">*</span></span>
        <input
          id="signup-displayname"
          bind:value={displayName}
          autocomplete="name"
          required
          maxlength="100"
          placeholder="Ada Lovelace"
        />
      </label>

      <!-- Username -->
      <label>
        <span>Username <span class="req">*</span></span>
        <input
          id="signup-username"
          bind:value={username}
          autocomplete="username"
          required
          minlength="3"
          maxlength="50"
          placeholder="ada_dev"
          class:field-error={touchedUsername && !usernameOk}
          class:field-ok={touchedUsername && usernameOk}
          onblur={() => (touchedUsername = true)}
        />
        {#if touchedUsername && !usernameOk}
          <span class="hint error-hint">3–50 chars · letters, numbers, _ and - only</span>
        {:else}
          <span class="hint">Letters, numbers, _ and - · no spaces</span>
        {/if}
      </label>

      <!-- Email -->
      <label>
        <span>Email <span class="req">*</span></span>
        <input
          id="signup-email"
          bind:value={email}
          type="email"
          autocomplete="email"
          required
          placeholder="ada@example.com"
        />
      </label>

      <!-- Password -->
      <label>
        <span>Password <span class="req">*</span></span>
        <input
          id="signup-password"
          bind:value={password}
          type="password"
          autocomplete="new-password"
          required
          minlength="8"
          placeholder="At least 8 characters"
          class:field-error={touchedPassword && !passwordOk}
          class:field-ok={touchedPassword && passwordOk}
          onblur={() => (touchedPassword = true)}
        />
        {#if password.length > 0}
          <div class="strength-bar">
            <div
              class="strength-fill"
              style="width:{passwordStr * 33.3}%; background:{passwordStrColor}"
            ></div>
          </div>
          <span class="hint" style="color:{passwordStrColor}">{passwordStrLabel}</span>
        {:else}
          <span class="hint">Minimum 8 characters</span>
        {/if}
      </label>

      <!-- Error -->
      {#if error}
        <p class="form-error" role="alert">⚠ {error}</p>
      {/if}

      <button class="btn-primary" type="submit" disabled={!canSubmit}>
        {loading ? 'Creating account…' : 'Create account'}
      </button>
    </form>

    <p class="switch">Already have an account? <a href="/login">Log in</a></p>
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
    width: min(100%, 460px);
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
    margin-bottom: 0;
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

  .req {
    color: #ef4444;
  }

  input {
    width: 100%;
    border: 1.5px solid var(--border);
    border-radius: 10px;
    background: var(--bg-card);
    color: var(--text-primary);
    font: inherit;
    padding: 0.85rem 1rem;
    transition: border-color 0.2s, outline 0.2s;
  }

  input:focus {
    border-color: var(--primary);
    outline: 3px solid rgba(99, 102, 241, 0.18);
  }

  input.field-error {
    border-color: #ef4444;
    outline: 3px solid rgba(239, 68, 68, 0.15);
  }

  input.field-ok {
    border-color: #22c55e;
  }

  .hint {
    font-size: 0.78rem;
    font-weight: 500;
    color: var(--text-muted);
  }

  .error-hint {
    color: #ef4444;
  }

  /* Password strength bar */
  .strength-bar {
    height: 4px;
    border-radius: 4px;
    background: var(--border);
    overflow: hidden;
    margin-top: 2px;
  }

  .strength-fill {
    height: 100%;
    border-radius: 4px;
    transition: width 0.3s ease, background 0.3s ease;
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
