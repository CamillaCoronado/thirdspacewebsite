<script>
    let email = '';
    let phone = '';
  </script>
  
  <main class="bg-indigo text-white font-sans">
    <!-- Navigation -->
    <header class="flex justify-between items-center p-16">
      <img src="../src/assets/logo-white.png" alt="thirdspace logo" class="h-10">
      <a href="/about" class="underline">About</a>
    </header>
  
    <!-- Main Section -->
    <section class="flex flex-col items-center">
      <div class="flex flex-col lg:flex-row items-center mb-32">
        <!-- Left: Screenshots -->
        <div class="flex flex-col md:flex-row justify-between items-center mb-16 lg:mb-0">
            <img src="../src/assets/onboarding-screenshots.png" alt="Map your path" class="mb-16">
        </div>
        
        <!-- Right: Waitlist Form -->
        <div class="bg-white text-indigo p-16 rounded-lg w-full max-w-lg">
          <h2 class="text-4xl font-bold mb-8">Introducing Thirdspace</h2>
          <p class="text-xl mb-16">Try this before opening up your marriage!</p>
          
          <form class="flex flex-col">
            <input 
              type="email" 
              placeholder="Email" 
              bind:value={email} 
              class="mb-16 px-8 py-4 rounded-md border border-gray-300"
            />
            <input 
              type="tel" 
              placeholder="Phone Number" 
              bind:value={phone} 
              class="mb-16 px-8 py-4 rounded-md border border-gray-300"
            />
            <p class="text-sm mb-16">We’ll only use your phone number to send a one-time text with the launch link. No spam, no sharing.</p>
            <button class="bg-indigo text-white px-8 py-4 rounded-full mb-16">Join waitlist</button>
          </form>
          
          <a href="/waitlist-check" class="underline">Already joined? Check waitlist number.</a>
        </div>
      </div>
  
      <!-- Astronaut Icon -->
      <div class="hidden md:block">
        <img src="../src/assets/astronaut.png" alt="Astronaut" class="h-64">
      </div>
    </section>
  </main>
  
  <style>
    /* Custom Tailwind overrides if needed */
  </style>
  