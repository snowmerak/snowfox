<script lang="ts">
  import "../app.postcss";
  import {
      ArrowKeyLeft,
      ArrowKeyRight,
      Button,
      ButtonGroup,
      DropdownDivider,
      Input,
      InputAddon,
      Listgroup
  } from "flowbite-svelte";
  import About from "$lib/reactive_bar/About.svelte";
  import {onMount} from "svelte";
  import {goto} from "$app/navigation";

  let scroll_y: number;
  let scroll_max_y: number;
  let scroll_gab: number = 0;
  const scroll_gab_max = 128;

  let left_side_reactive_menu = {
      "/about": About,
  };

  let right_side_reactive_menu = {
  }

  let left_side_reactive_action = {
  };

  let right_side_reactive_action = {
  };

  let menu_list = [
      { name: "Home", url: "/" },
      { name: "About", url: "/about" },
      { name: "Contact", url: "/contact" },
  ];

  let searched_menu_list = [];

  let pathname = "";
  let pathname_without_query = "";

  onMount(() => {
      pathname = window.location.pathname;
      pathname_without_query = pathname.split("?")[0];

      scroll_max_y = document.body.scrollHeight - window.innerHeight;
  });

  $: scroll_gab = scroll_max_y - scroll_y;
</script>

<svelte:window bind:scrollY={scroll_y}></svelte:window>

<div>
    <div class="flex flex-col">
        <ButtonGroup class="w-full space-x-px">
            <svelte:component this={left_side_reactive_menu[pathname_without_query]} />
            <Input type="text" on:keyup={(ev) => {
                let value = ev.target.value;
                if (value === "") {
                    searched_menu_list = [];
                    return;
                }
                let regex = new RegExp(value, 'iu');
                searched_menu_list = menu_list.filter((item) => {
                    return regex.test(item.name);
                }).map((item) => {
                    let current = false;
                    let pathname_without_query = window.location.pathname.split("?")[0];
                    if (pathname_without_query === item.url) {
                        current = true;
                    }
                    return {
                        name: item.name,
                        url: item.url,
                        current: current,
                    }
                });
            }} placeHolder="Search Menu"></Input>
            <svelte:component this={right_side_reactive_menu[pathname]} />
        </ButtonGroup>
        <DropdownDivider></DropdownDivider>
        {#if searched_menu_list.length > 0 }
            <Listgroup active items={searched_menu_list} let:item on:click={(ev) => {
                goto(ev.detail.url);
            }}>{item.name}</Listgroup>
            <DropdownDivider></DropdownDivider>
        {/if}
    </div>
    <div class="grow">
        <slot />
    </div>
<!--    <div class="fixed bottom-0 left-0 w-full">-->
    <div class="bottom-0 left-0 w-full" class:fixed={scroll_gab > scroll_gab_max}>
        <DropdownDivider></DropdownDivider>
        <ButtonGroup class="w-full space-x-px">
            <svelte:component this={left_side_reactive_action[pathname_without_query]} />
            <Input></Input>
            <svelte:component this={right_side_reactive_action[pathname]} />
        </ButtonGroup>
    </div>
</div>
