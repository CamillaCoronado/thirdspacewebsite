<script>
    let email = '';
    let phone = '';
  </script>
  
  <main class="bg-indigo text-white px-16">
    <!-- Navigation -->
    <header class="flex justify-between flex-col md:flex-row items-center p-16 md:mb-64">
      <a href="/about" class="underline text-white self-end md:self-center text-base mb-32 md:mb-0 grow md:grow-0 md:order-2">About</a>
      <img src="../src/assets/logo-white.png" alt="thirdspace logo" class="w-[262px] md:w-[initial] md:h-[40px] md:order-1">
    </header>
  
    <!-- Main Section -->
    <section class="flex flex-col items-center">
      <div class="flex flex-col lg:flex-row lg:justify-center items-center w-full gap-[16px]">
        <!-- Right: Waitlist Form -->
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
            <button class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Join waitlist</button>
          </form>
          
          <a href="/waitlist-check" class="underline text-white mt-[-8px] block text-center mb-32">Already joined? Check waitlist number.</a>
        </div>
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

  