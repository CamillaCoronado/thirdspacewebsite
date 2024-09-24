<script>
    let email = '';
    let phone = '';
  </script>
  
  <main class="bg-indigo text-white px-16">
    <!-- Navigation -->
    <header class="flex justify-between flex-col items-center p-16">
      <a href="/about" class="underline text-white self-end text-base mb-32 grow">About</a>
      <img src="../src/assets/logo-white.png" alt="thirdspace logo" class="h-[55px] w-[274px]">
    </header>
  
    <!-- Main Section -->
    <section class="flex flex-col items-center">
      <div class="flex flex-col lg:flex-row items-center mb-32">
        <!-- Right: Waitlist Form -->
        <div class="text-white rounded-lg w-full max-w-lg flex-col">
          <h4 class="text-2xl font-bold mb-8 uppercase hidden md:block">Introducing Thirdspace</h4>
          <h2 class="text-[40px] mb-32 font-bold leading-[49px]">presto chango you now have a friendo</h2>
          
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
            <p class="text-sm mb-16 text-bold text-white">We’ll only use your phone number to send a one-time text with the launch link. No spam, no sharing.</p>
            <button class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Join waitlist</button>
          </form>
          
          <a href="/waitlist-check" class="text-bold underline text-white mt-[-8px] block text-center">Already joined? Check waitlist number.</a>
        </div>
        <div class=" screenshots flex flex-row flex-wrap md:flex-row justify-between items-center mb-16 lg:mb-0">
          <img src="../src/assets/onboarding-page-1.png" alt="Map your path" class="mb-16 w-[175px]">
          <img src="../src/assets/onboarding-page-2.png" alt="Instant Match" class="mb-16 w-[175px]">
          <img src="../src/assets/onboarding-page-3.png" alt="Meet and Share" class="mb-16 w-[175px]">
          <img src="../src/assets/astronaut.png" alt="Astronaut" class="object-contain h-[240px] w-[173px]">
      </div>
      </div>
    </section>
  </main>
  