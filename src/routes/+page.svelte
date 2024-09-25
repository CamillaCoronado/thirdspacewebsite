<script lang="ts">
    let email = '';
    let phone = '';
    let waitlistLink: string = "https://jointthirdspace.com/somequ";
    let waitlistNumber: number = 432;

    let showNewBlock: string = "initial";

    function handleSubmit() {
    // Process the input fields
    console.log('Processing:', email, phone);

    // After processing, replace the block
    showNewBlock = "waitlist success";
  }

  function copyToClipboard() {
    navigator.clipboard.writeText(waitlistLink);
    alert("Link copied to clipboard!"); // Optional alert
  }

  function waitlistCheck() {
    showNewBlock = "waitlist check";
  }
  function waitlistCheckSuccess() {
    showNewBlock = "waitlist check success";
  }

 function backJoin() {
  showNewBlock = "initial"
 }
  </script>
  
  <main class="bg-indigo text-white px-16 pt-16 min-h-[100vh]">
    <!-- Navigation -->
    <header class="flex justify-between flex-col md:flex-row items-center md:justify-end mb-32 md:mb-64">
      <!-- <a href="/about" class="underline text-white self-end md:self-center text-base mb-32 md:mb-0 grow md:grow-0 md:order-2">About</a> -->
      <img on:click={backJoin} src="../src/assets/logo-white.png" alt="thirdspace logo" class="w-[initial] h-[40px] md:order-1 cursor-pointer">
    </header>
  
    <!-- Main Section -->
    <section class="flex flex-col items-center">
      <div class="mb-32 flex flex-col lg:flex-row lg:justify-center items-center w-full gap-[32px] md:gap-[16px]">
        {#if showNewBlock == "initial"} <!-- Right: Waitlist Form -->
        <div class="text-white rounded-lg w-full max-w-lg flex-col">
          <h4 class="text-2xl font-bold mb-8 uppercase hidden md:block">Introducing Thirdspace</h4>
          <h2 class="text-[40px] mb-16 font-bold leading-[49px]">presto chango you now have a friendo</h2>
          
          <form class="flex flex-col text-dark-charcoal">
            <input 
              type="email" 
              placeholder="Email" 
              bind:value={email} 
              class="mb-16 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
            />
            <input 
              type="tel" 
              placeholder="Phone Number" 
              bind:value={phone} 
              class="mb-8 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
            />
            <p class="text-sm mb-16 bold-text text-white">We’ll only use your phone number to send a one-time text with the launch link. No spam, no sharing.</p>
            <button on:click={handleSubmit} class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Join waitlist</button>
          </form>
          
          <button on:click={waitlistCheck} class="underline bold-text text-sm text-white mt-[-8px] block text-center mb-32">Already joined? Check waitlist number.<button/>
        </div>
        {:else if showNewBlock == "waitlist success"}
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left">
          <h2 class="font-bold mb-16">Yay! You're on the waitlist.</h2>
          <p class="mb-16 bold-text text-lg">Skip ahead in line by referring friends. Top 15 get one month premium free.</p>
        
          <div class="flex justify-center items-center mb-16 space-x-2">
            <!-- Input field -->
            <input 
              type="text" 
              placeholder="https://jointthirdspace.com/somequ"
              bind:value={waitlistLink}
              readonly 
              class="text-xl block w-full px-8 py-8 border-2 border-medium-indigo text-dark-charcoal rounded-md"
            />
            <!-- Copy button (icon) -->
            <button on:click={copyToClipboard}>
              <img src="../src/assets/copy-icon.png" alt="Copy" class="w-[32px] h-[34px]" />
            </button>
          </div>
        
          <!-- Social media icons -->
          <div class="flex justify-center gap-[16px] mb-8">
            <a href="#" class="block"><img src="../src/assets/instagram-logo.png" alt="Instagram" class="w-10 h-10" /></a>
            <a href="#" class="block"><img src="../src/assets/twitter-logo.png" alt="X (Twitter)" class="w-10 h-10" /></a>
            <a href="#" class="block"><img src="../src/assets/facebook-logo.png" alt="Facebook" class="w-10 h-10" /></a>
          </div>
        
          <!-- Waitlist Number -->
          <p class="text-base text-center bold-text">Waitlist Number: {waitlistNumber}</p>
        </div>
        {:else if showNewBlock == "waitlist check"}
        <form class="flex flex-col max-w-lg text-dark-charcoal">
          <p class="mb-16 bold-text text-lg text-white">Please provide the email or phone number you joined with.</p>
          <input 
            type="email" 
            placeholder="Email" 
            bind:value={email} 
            class="mb-8 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
          />
          <p class="mb-8 bold-text text-white text-lg text-center">Or</p>
          <input 
            type="tel" 
            placeholder="Phone Number" 
            bind:value={phone} 
            class="mb-16 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
          />
          <button on:click={waitlistCheckSuccess} class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Check waitlist</button>
          <button on:click={backJoin} class="text-base block underline text-white text-left mt-[-8px] bold-text">Join waitlist</button>
        </form>
        {:else if showNewBlock == "waitlist check success"}
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left">
          <h2 class="font-bold mb-16">Yay! You're {waitlistNumber} on the waitlist.</h2>
          <p class="mb-16 bold-text text-lg">Skip ahead in line by referring friends. Top 15 get one month premium free.</p>
        
          <div class="flex justify-center items-center mb-16 space-x-2">
            <!-- Input field -->
            <input 
              type="text" 
              placeholder="https://jointthirdspace.com/somequ"
              bind:value={waitlistLink}
              readonly 
              class="text-xl block w-full px-8 py-8 border-2 border-medium-indigo text-dark-charcoal rounded-md"
            />
            <!-- Copy button (icon) -->
            <button on:click={copyToClipboard}>
              <img src="../src/assets/copy-icon.png" alt="Copy" class="w-[32px] h-[34px]" />
            </button>
          </div>
        
          <!-- Social media icons -->
          <div class="flex justify-center gap-[16px] mb-8">
            <a href="#" class="block"><img src="../src/assets/instagram-logo.png" alt="Instagram" class="w-10 h-10" /></a>
            <a href="#" class="block"><img src="../src/assets/twitter-logo.png" alt="X (Twitter)" class="w-10 h-10" /></a>
            <a href="#" class="block"><img src="../src/assets/facebook-logo.png" alt="Facebook" class="w-10 h-10" /></a>
          </div>
        </div>
        {/if}
        <div class="flex flex-row flex-wrap md:flex-row justify-center items-start mb-16 lg:mb-0">
          <img src="../src/assets/onboarding-page-1.png" alt="Map your path" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px]">
          <img src="../src/assets/onboarding-page-2.png" alt="Instant Match" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px]">
          <div class= "grow flex justify-center">
            <img src="../src/assets/onboarding-page-3.png" alt="Meet and Share" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px] flex-center">
          </div>
        </div>
      </div>
    </section>
  </main>
  <img src="../src/assets/astronaut.png" alt="Astronaut" class="astronaut transition duration-1000 ease-in-out object-contain h-[78px] w-[59px]">

  