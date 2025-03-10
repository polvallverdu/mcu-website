<script lang="ts">
    import "../app.css"

    import { SvelteToast } from '@zerodevx/svelte-toast'

    import type { LayoutData } from "./$types"
    export let data: LayoutData

    import { Favourites, UtilList } from "$lib/store"
    import { onMount } from "svelte";

    UtilList.set(data.utils)

    let currentUrl = "";

    // Load favourites from cookies on page load
    onMount(() => {
        currentUrl = window.location.pathname.substring(1);

        const cookies = document.cookie.split(";")
        let favourites: string[] = []
        for (const i in cookies) {
            if (cookies[i].includes("mcu-favourites=")) favourites = cookies[i].split("=")[1].split("//")
        }
        Favourites.set(favouriteUtilsFromArray(favourites))
        UtilList.set(data.utils.filter((util: { path: string }) => !favourites.includes(util.path)))
    })

    const updateFavourites = (path: string) => {
        const cookies = document.cookie.split(";")
        let favourites: string[] = []
        for (const i in cookies) {
            if (cookies[i].includes("mcu-favourites=")) favourites = cookies[i].split("=")[1].split("//")
        }

        if (favourites.includes(path)) {
            const index = favourites.indexOf(path)
            favourites.splice(index, 1) // Remove from array 
        } else {
            favourites.push(path)
        }

        Favourites.set(favouriteUtilsFromArray(favourites))
        UtilList.set(data.utils.filter((util: { path: string }) => !favourites.includes(util.path)))

        const date = new Date()
        date.setTime(date.getTime() + 360 * 24 * 60 * 60 * 1000) // Add 360 days
        document.cookie = `mcu-favourites=${favourites.join("//")};Expires=${date.toUTCString()};Secure;SameSite=Lax;Path=/`
    }

    const favouriteUtilsFromArray = (favourites: string[]) => {
        return data.utils.filter((util: { path: string }) => favourites.includes(util.path))
    }

    const filterUtils = (query: string) => {
        UtilList.set(data.utils.filter((util: { name: string }) => util.name.toLowerCase().includes(query.toLowerCase())))
    }

    let searchValue: string

    const handleKeyPress = (event: KeyboardEvent) => {
        if (event.key === "Enter") filterUtils(searchValue)
    }

    const handleInput = () => {
        filterUtils(searchValue)
    }

    let navShown = false
</script>

<button on:click={() => navShown = !navShown} class="lg:hidden absolute top-8 left-8 {navShown ? '' : ''}">
    <img src="/icon/{navShown ? 'x.svg' : 'hamburger.svg'}" alt="Hamburger" class="h-7">
</button>

<div class="flex lg:flex-row flex-col">
    <div class="{navShown ? 'block' : 'hidden'} lg:block lg:sticky lg:left-0 lg:top-0 lg:h-screen lg:min-w-[22rem] lg:max-w-[22rem] lg:min-h-screen bg-[#1D1F24]">
        <div class="flex flex-col h-full ">
            <div class="flex w-full items-center flex-col space-y-4 mt-4 lg:mt-10 mb-6">
                <div class="absolute"></div>
                <a href="/" on:click={() => currentUrl = "/"} aria-label='Homepage'><img src="/img/logo.svg" alt="" class="h-11"></a>
                <div class="search h-10 text-md lg:w-[80%] w-[290px]">
                    <img src="/icon/search.svg" alt="Search Icon" class="h-5">
                    <input bind:value={searchValue} on:input={handleInput} type="text" placeholder="Search..." on:keypress={handleKeyPress} on:blur={handleInput} class="w-full pt-0.5 pr-0.5 justify-center items-center self-center">
                </div>
            </div>
            <div class="overflow-y-scroll overflow-x-hidden h-[200px] flex-1 mb-10 lg:mb-0 mx-auto lg:mx-0">
                <div data-sveltekit-reload class="px-4">
                    {#if $Favourites.length > 0}
                        <div class="flex flex-col space-y-2 mb-6 ">
                            <h2 class="text-[#626875] uppercase font-bold text-sm px-2">Favorites</h2>
                            {#each $Favourites as util}
                                <div class="flex items-center justify-between px-2 {util.path === currentUrl ? 'py-0.5 bg-[#212429] rounded-lg' : ''}">
                                    <a href="{util.path}" aria-label='{util.path}' class="flex space-x-4">
                                        <img src="/component/icon/{util.path}.svg" alt="" class="h-9 self-center">
                                        <div class="flex flex-col">
                                            <h2 class="text-[#AEB2BC] font-semibold text-[20px]">{util.name}</h2>
                                            <p class="text-[#3C414B]">{@html util.shortDescription}</p>
                                        </div>
                                    </a>
                                    <!-- svelte-ignore a11y-click-events-have-key-events -->
                                    <svg class="w-5 h-5 fill-[#FF5B5B] cursor-pointer hover:scale-110 transition-transform duration-200" on:click={() => updateFavourites(util.path)} viewBox="0 0 512 512"><!--! Font Awesome Pro 6.4.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. --><path d="M47.6 300.4L228.3 469.1c7.5 7 17.4 10.9 27.7 10.9s20.2-3.9 27.7-10.9L464.4 300.4c30.4-28.3 47.6-68 47.6-109.5v-5.8c0-69.9-50.5-129.5-119.4-141C347 36.5 300.6 51.4 268 84L256 96 244 84c-32.6-32.6-79-47.5-124.6-39.9C50.5 55.6 0 115.2 0 185.1v5.8c0 41.5 17.2 81.2 47.6 109.5z"/></svg>
                                </div>
                            {/each}
                        </div>
                        {#if $UtilList.length > 0}
                            <h2 class="text-[#626875] uppercase font-bold text-sm mb-2 px-2">Others</h2>
                        {/if}
                    {/if}
                    <div class="flex flex-col space-y-1">
                        {#each $UtilList as util}
                            <div class="flex items-center justify-between px-2 {util.path === currentUrl ? 'py-0.5 bg-[#212429] rounded-lg' : ''}">
                                <a href="{util.path}" aria-label='{util.path}' class="flex space-x-4">
                                    <img src="/component/icon/{util.path}.svg" alt="" class="h-9 self-center">
                                    <div class="flex flex-col">
                                        <h2 class="text-[#AEB2BC] font-semibold text-[20px]">{util.name}</h2>
                                        <p class="text-[#3C414B]">{@html util.shortDescription}</p>
                                    </div>
                                </a>
                                <!-- svelte-ignore a11y-click-events-have-key-events -->
                                <svg class="w-5 h-5 fill-[#3C414B] hover:fill-[#FF5B5B] cursor-pointer hover:scale-110 transition-transform duration-200" on:click={() => updateFavourites(util.path)} viewBox="0 0 512 512"><!--! Font Awesome Pro 6.4.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. --><path d="M225.8 468.2l-2.5-2.3L48.1 303.2C17.4 274.7 0 234.7 0 192.8v-3.3c0-70.4 50-130.8 119.2-144C158.6 37.9 198.9 47 231 69.6c9 6.4 17.4 13.8 25 22.3c4.2-4.8 8.7-9.2 13.5-13.3c3.7-3.2 7.5-6.2 11.5-9c0 0 0 0 0 0C313.1 47 353.4 37.9 392.8 45.4C462 58.6 512 119.1 512 189.5v3.3c0 41.9-17.4 81.9-48.1 110.4L288.7 465.9l-2.5 2.3c-8.2 7.6-19 11.9-30.2 11.9s-22-4.2-30.2-11.9zM239.1 145c-.4-.3-.7-.7-1-1.1l-17.8-20c0 0-.1-.1-.1-.1c0 0 0 0 0 0c-23.1-25.9-58-37.7-92-31.2C81.6 101.5 48 142.1 48 189.5v3.3c0 28.5 11.9 55.8 32.8 75.2L256 430.7 431.2 268c20.9-19.4 32.8-46.7 32.8-75.2v-3.3c0-47.3-33.6-88-80.1-96.9c-34-6.5-69 5.4-92 31.2c0 0 0 0-.1 .1s0 0-.1 .1l-17.8 20c-.3 .4-.7 .7-1 1.1c-4.5 4.5-10.6 7-16.9 7s-12.4-2.5-16.9-7z"/></svg>
                            </div>
                        {/each}
                    </div>
                </div>
            </div>
            <div class="bg-[#1B1D21] hidden lg:block ">
                <a href="https://discord.gg/CGmMQwfXXN" aria-label='Discord' target="_blank" class="w-full self-center px-4 text-[#3C414B] text-sm text-center">
                    <p>Got a question? Even if you don’t, click to join our awesome community on <span class="underline decoration-dotted">Discord</span>!</p>
                </a>
            </div>
        </div>
    </div>
    <div class="flex flex-col justify-start items-center w-full mb-20">
        <slot/>
    </div>
</div>

<style>


    ::-webkit-scrollbar{
        width: 7px;
    }

    ::-webkit-scrollbar-track-piece{
        background-color: #1d1f24;
    }
</style>

<SvelteToast />
