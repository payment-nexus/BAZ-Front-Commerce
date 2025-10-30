<script>
  /* stores */
  import {
    isLoggedIn,
    loggedInUser,
    linkSelected,
    sidebar,
  } from "$lib/stores.js";
  /* controllers */
  import { AuthController } from "$lib/controllers/auth/auth.controller";
  /* router */
  import { optionsSidebar } from "$lib/hooks/router.js";
  /* assets */
  import logo from "$lib/assets/Logo.png";
  import noUser from "$lib/assets/no_user.png";
  /* components */
  import Icons from "$lib/components/Icons.svelte";
  import ThemeToggle from "$lib/components/ThemeToggle.svelte";
  /* utils */
  import { cutEmail } from "$lib/utils/string";

  let options = [];
  let innerWidth;
  let innerHeight;

  $: {
    if ($isLoggedIn) {
      optionsSidebar($loggedInUser.accountType).then((response) => {
        options = response;
      });
      if (innerWidth <= 1000) {
        $sidebar = false;
      } else {
        $sidebar = true;
      }
    }
  }
</script>

<svelte:window bind:innerWidth bind:innerHeight />
<div class="navbar no-print">
  <i
    on:click={() => ($sidebar = !$sidebar)}
    on:keydown={() => ($sidebar = !$sidebar)}
  >
    <Icons name="menu-lines" width="24" height="24" />
  </i>
  <div class="logo">
    <a href="/">
      <img src={logo} alt="Company Logo" />
    </a>
  </div>
  <div class="theme-toggle">
    <ThemeToggle />
  </div>
</div>
<div class="sidebar no-print {$sidebar ? '' : 'close'}">
  <div class="sidebar-start">
    <div class="logo-details">
      <i
        on:click={() => ($sidebar = !$sidebar)}
        on:keydown={() => ($sidebar = !$sidebar)}
      >
        <Icons name="menu-lines" width="24" height="24" />
      </i>
      {#if innerWidth >= 1000}
        <a href="/">
          <div class="text-logo">
            <div class="title">
              <p>WaliApp</p>
            </div>
          </div>
          <img src={logo} alt="Company Logo" />
        </a>
      {/if}
    </div>
    <!-- //TODO Return href to /profile when profile page is done -->
    <a
      on:click={() => {
        $linkSelected = "Perfil";
      }}
      on:keypress={(e) =>
        e.key === "Enter"
          ? ($linkSelected = "Perfil")
          : ($linkSelected = $linkSelected)}
      href="/#"
      class="profile-details"
    >
      <div class="profile-content">
        <img src={$loggedInUser?.avatar ?? noUser} alt="profileImg" />
      </div>
      <div class="name-job">
        <div class="profile_name">
          {$loggedInUser?.name ??
            cutEmail($loggedInUser?.email.toUpperCase()) ??
            ""}
        </div>
        <div class="job">
          {$loggedInUser?.businessName ?? cutEmail($loggedInUser?.email) ?? ""}
        </div>
      </div>
    </a>
    <ul class="nav-links">
      {#each options as option}
        <li
          on:click={() => {
            $linkSelected = option.name;
          }}
          on:keypress={(e) =>
            e.key === "Enter"
              ? ($linkSelected = option.name)
              : ($linkSelected = $linkSelected)}
          class={$linkSelected === option.name ? "active-link_name" : ""}
        >
          <a href={option.path}>
            <i class={$linkSelected === option.name ? "active-link_name" : ""}>
              <Icons name={option.icon} width="24" height="24" />
            </i>
            <span
              class={$linkSelected === option.name
                ? "active-link_name"
                : "link_name"}>{option.name}</span
            >
          </a>
          <ul class="sub-menu blank">
            <li><a class="link_name" href={option.path}>{option.name}</a></li>
          </ul>
        </li>
      {/each}
    </ul>
  </div>
  <div class="sidebar-bottom">
    <ul class="nav-links">
      <div class="theme-toggle">
        <ThemeToggle />
      </div>
      <!-- <li
        on:click={() => {
          $linkSelected = "Ayuda";
        }}
        on:keypress={(e) => (e.key === "Enter" ? ($linkSelected = "Ayuda") : ($linkSelected = $linkSelected))}
      >
        <a href="/help">
          <i class={$linkSelected === "Ayuda" ? "active-link_name" : ""}>
            <Icons name="help" width="24" height="24" />
          </i>
          <span class={$linkSelected === "Ayuda" ? "active-link_name" : "link_name"}>Ayuda</span>
        </a>
        <ul class="sub-menu blank">
          <li><a class="link_name" href="/help">Ayuda</a></li>
        </ul>
      </li> -->
      <li class="logout">
        <a
          href="/login"
          on:click={async () => {
            await AuthController.logout($loggedInUser.$id);
          }}
        >
          <i>
            <Icons name="logout-box-line" width="24" height="24" />
          </i>
          <span class="link_name">Cerrar Sesión</span>
        </a>
        <ul class="sub-menu blank">
          <li>
            <a
              class="link_name"
              href="/login"
              on:click={async () => {
                await AuthController.logout($loggedInUser.$id);
              }}>Cerrar Sesión</a
            >
          </li>
        </ul>
      </li>
    </ul>
  </div>
</div>

<style lang="scss">
  @import "src/lib/styles/sidebar.scss";
</style>
