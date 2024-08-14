<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import '$lib/app.css'; 
  let email: string = '';
  let password: string = '';
  let error: string | null = null;
  let loggedInUser: string | null = null; 
  
  let categories: any = []; 

 
  async function fetchCategories() {
    try {
      let { data, error } = await supabase
        .from('category') 
        .select('*'); 
      if (error) {
        console.error('Error fetching data:', error.message);
        return;
      }

      categories = data;
    } catch (err) {
      console.error('Unexpected error:', err);
    }
  }

  
  async function handleSignIn() {
    try {
      console.log('Email:', email);
      console.log('Password:', password);

      const { data, error: signInError } = await supabase.auth.signInWithPassword({
        email,
        password
      });
  
      if (signInError) {
        console.error('Error during sign-in:', signInError.message);
        error = signInError.message; 
        return;
      }
  
      console.log('User signed in successfully');
      loggedInUser = data.user?.email || null; 
      error = null; 

      
      await fetchCategories();

    } catch (err) {
      console.error('Unexpected error:', err);
      error = 'Sign-in failed. Please try again.';
    }
  }

  async function handleSignOut() {
    try {
      const { error: signOutError } = await supabase.auth.signOut();

      if (signOutError) {
        console.error('Error during sign-out:', signOutError.message);
        error = signOutError.message; 
        return;
      }

      console.log('User signed out successfully');
      loggedInUser = null; 
      error = null; 

      categories = [];
    } catch (err) {
      console.error('Unexpected error:', err);
      error = 'Sign-out failed. Please try again.';
    }
  }
</script>


<div class="login-form">
  <h2>Login</h2>
  {#if loggedInUser}
    <p>{loggedInUser} is successfully logged in.</p> 
    <button on:click={handleSignOut}>Sign Out</button>
  {:else}
    <form on:submit|preventDefault={handleSignIn}>
      <label for="email">Email</label>
      <input type="email" id="email" bind:value={email} required>

      <label for="password">Password</label>
      <input type="password" id="password" bind:value={password} required>

      <button type="submit">Sign In</button>
    </form>
  {/if}
  {#if error}
    <p style="color: red;">{error}</p>
  {/if}
</div>


{#if loggedInUser && categories.length > 0}
  <div class="category-list">
    <h2>Categories</h2>
    <ul>
      {#each categories as category}
        <li>
          <strong>{category.c_name}</strong><br>
          <em>{category.c_description}</em><br>
          <small>Email: {category.email}</small><br>
          <small>Password: {category.password}</small>
        </li>
      {/each}
    </ul>
  </div>
{:else if loggedInUser}
  <p>No categories found.</p>
{/if}
