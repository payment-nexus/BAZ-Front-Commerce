<script>
  /* svelte */
  import { goto } from "$app/navigation";
  import { onMount } from "svelte";
  /* store */
  import { isLoggedIn } from "$lib/stores";
  /* components */
  import Input from "$lib/components/Input.svelte";
  import PasswordInput from "$lib/components/inputs/PasswordInput.svelte";
  import Loader from "$lib/components/Loader.svelte";
  /* controllers */
  import { AuthController } from "$lib/controllers/auth/auth.controller";
  // import { AmexController } from "$lib/controllers/amex/amex.controller";
  /* utils */
  import { validateEmail, validatePassword } from "$lib/utils/input";
  /* assets */
  import Logo from "$lib/assets/Logo.png";
  /* const */
  // import { amexData, amexAuthorization } from "$lib/constants/amex";

  /* consts */
  const input = {
    email: "",
    password: "",
  };

  /* dynamic vars */
  let error = false;
  let loading = true;

  /* handlers & functions */
  async function handleLogin() {
    loading = true;
    const data = await AuthController.login(input);
    loading = false;
    if (data?.error) error = data.error;
  }

  /* async function amexNewCommerce() {
    const data = await AmexController.newCommerce(amexData, amexAuthorization);
  } */

  onMount(async () => {
    // amexNewCommerce();
    if ($isLoggedIn) await goto("/");
    loading = false;
  });
</script>

<div class="container">
  {#if loading}
    <Loader />
  {:else}
    <div class="content">
      <div class="logo">
        <img src={Logo} alt="Company Logo" />
        <div class="text">
          <p>WaliApp</p>
        </div>
      </div>
      <div class="form">
        <div class="title">Inicio de Sesión</div>
        <div class={`subtitle ${!error ? "hidden" : ""}`}>
          Verifica que tus datos sean correctos
        </div>
        <div class="form-inputs">
          <form on:submit|preventDefault={handleLogin}>
            <Input
              label="Correo Electrónico"
              id="login-email"
              bind:value={input.email}
              name="email"
              type="email"
              className={`txt-field ${!error ? "normal" : "invalid"}`}
              placeholder="ejemplo@correo.com"
            />
            <PasswordInput
              label="Contraseña"
              id="login-password"
              bind:value={input.password}
              name="password"
              className={`txt-field ${!error ? "normal" : "invalid"}`}
              placeholder="contraseña"
            />
            <div class="forgot-pass-link">
              <a href="/update-pass">Olvidé mi Contraseña</a>
            </div>
            <div class="btn-layout">
              <Input
                label="Iniciar Sesión"
                id="loginButton"
                type="submit"
                className={validateEmail(input.email) &&
                validatePassword(input.password)
                  ? "btn"
                  : "btn-disabled"}
                icon=""
              />
            </div>
            <!-- <div class="register-link">
              <a href="/register">Quiero crear una cuenta</a>
            </div> -->
          </form>
        </div>
      </div>
    </div>
  {/if}
</div>

<style lang="scss">
  @import "src/lib/styles/login.scss";
</style>
