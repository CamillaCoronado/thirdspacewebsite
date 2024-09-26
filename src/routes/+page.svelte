<script lang="ts">
  import { firestore } from '$lib/utils/firebaseSetup';
  import { collection, addDoc, query, where, getDocs, getDoc, setDoc, doc, updateDoc, increment } from 'firebase/firestore';
  import { v4 as uuidv4 } from 'uuid';

  let email = '';
  let phone = '';
  let waitlistLink: string = "";
  let populateWaitlistInfo: number;

  let showNewBlock: string = "initial";

    
export const handleReferral = async (uniqueID: string) => {
  const referrerRef = doc(firestore, 'waitlist', uniqueID);
};


  const handleSubmit = async (e: Event) => {
  e.preventDefault();
  
  try {
    // Add user to waitlist
    const result = await addToWaitlist(email, phone);

    // Handle waitlist success
    handleWaitlistAction("waitlistSuccess");
    const success = true;
    console.log("Successfully added to waitlist:", result);

    // Check for a referral in the URL (if a uniqueID is present)
    const urlParams = new URLSearchParams(window.location.search);
    const uniqueID = urlParams.get('referral'); // Assuming the referrer's unique ID is passed as a query param

    if (uniqueID) {
      // Handle referral if there is a uniqueID
      await handleReferral(uniqueID);
      console.log("Referral processed for ID:", uniqueID);
    }

  } catch (error) {
    // Handle error if addToWaitlist fails
    console.error("Error adding to waitlist:", error);
    const success = false;
  }
};


const getNextWaitlistNumber = async () => {
  try {
    // Step 1: Log when the function starts
    console.log("getNextWaitlistNumber called");

    const metaDocRef = doc(firestore, 'meta', 'waitlistTracker');
    console.log("metaDocRef created:", metaDocRef.path);

    // Step 2: Try to get the document from Firestore
    const metaDoc = await getDoc(metaDocRef);
    console.log("metaDoc fetched:", metaDoc.exists() ? "Document exists" : "Document does not exist");

    // Step 3: Check if the document exists and log the current data
    if (metaDoc.exists()) {
      const data = metaDoc.data();
      console.log("Document data:", data);

      const currentNumber = data?.currentWaitlistNumber || 0;
      console.log("Current waitlist number:", currentNumber);

      // Step 4: Update the document with the new waitlist number
      await updateDoc(metaDocRef, {
        currentWaitlistNumber: currentNumber + 1,
      });
      console.log("Waitlist number incremented to:", currentNumber + 1);

      return currentNumber + 1;
    } else {
      // Step 5: Handle case where the document doesn't exist
      console.log("Document does not exist, creating it with currentWaitlistNumber set to 1");

      await setDoc(metaDocRef, { currentWaitlistNumber: 1 });
      console.log("Document created with waitlist number 1");

      return 1;
    }
  } catch (error) {
    // Step 6: Log any errors
    console.error("Error in getNextWaitlistNumber:", error);
    throw error;
  }
};

  export const incrementReferralCount = async (uniqueId: string) => {
  try {
    const waitlistRef = collection(firestore, 'waitlist');
    const q = query(waitlistRef, where('unique_link', '==', `https://jointhirdspace.com/invite?referral=${uniqueId}`));
    const querySnapshot = await getDocs(q);

    if (!querySnapshot.empty) {
      const docRef = querySnapshot.docs[0].ref;
      await updateDoc(docRef, {
        referral_count: increment(1),
      });
    }
  } catch (error) {
    console.error('Error updating referral count: ', error);
    throw error;
  }
};

export const getReferralCount = async () => {
  try {
    // Reference to the 'waitlist' collection
    const waitlistRef = collection(firestore, 'waitlist');
    const urlParams = new URLSearchParams(window.location.search);
    const uniqueID = urlParams.get('referral');

    // Query to find the document by unique_link
    const q = query(waitlistRef, where('unique_link', '==', `https://jointhirdspace.com/invite?referral=${uniqueID}`));
    const querySnapshot = await getDocs(q);

    if (!querySnapshot.empty) {
      // Retrieve the referral_count from the first matching document
      const userDoc = querySnapshot.docs[0];
      const userData = userDoc.data();
      
      // Return the referral_count
      console.log("referral count: " + userData.referral_count);
      return userData.referral_count;
    } else {
      throw new Error('Document not found');
    }
  } catch (error) {
    console.error('Error getting referral count: ', error);
    throw error;  // Re-throw the error to be handled by the calling code
  }
};

  function handleWaitlistAction(action: string) {
  switch (action) {
    case 'copyToClipboard':
      navigator.clipboard.writeText(waitlistLink);
      alert("Link copied to clipboard!"); // Optional alert
      break;
    case 'waitlistSuccess':
      displayWaitlistInfo();
      showNewBlock = "waitlist success";
      break;
    case 'waitlistCheck':
      showNewBlock = "waitlist check";
      break;
    case 'waitlistCheckSuccess':
      displayWaitlistInfo();
      showNewBlock = "waitlist check success";
      break;
    case 'initial':
      showNewBlock = "initial";
      break;
    default:
      console.error("Unknown action");
  }
}

const generateUniqueLink = (): string => {
  const uniqueId = uuidv4();
  return `https://jointhirdspace.com/invite?referral=${uniqueId}`;
};

export const addToWaitlist = async (email: string, phone: string) => {
  try {
    const uniqueLink = generateUniqueLink();
    const waitlistRef = collection(firestore, 'waitlist');
    const waitlistNumber = await getNextWaitlistNumber();
    const referralCount = 0;
    await addDoc(waitlistRef, {
      email: email,
      phone: phone,
      waitlist_number: waitlistNumber,
      unique_link: uniqueLink,
      referral_count: referralCount,
    });
    console.log('Document successfully written!');
    return true; // Indicate success
  } catch (error) {
    console.error('Error writing document: ', error);
    throw error; // Re-throw the error to be caught by handleSubmit
  }
};

export const getWaitlistInfo = async (email: string, phone: string) => {
  try {
    // Query Firestore to find the user by email or phone
    const waitlistRef = collection(firestore, 'waitlist');
    const q = email ? query(waitlistRef, where('email', '==', email)) : query(waitlistRef, where('phone', '==', phone));
    console.log("email: " + email);
    console.log("phone: " + phone);
    const querySnapshot = await getDocs(q);
    
    if (!querySnapshot.empty) {
      // Assuming only one document matches, get the first result
      const userDoc = querySnapshot.docs[0];
      const userData = userDoc.data();

      // Extract the unique URL and waitlist number
      const uniqueUrl = userData.unique_link;
      const waitlistNumber = userData.waitlist_number;

      // Return the data to be used in the UI
      return { uniqueUrl, waitlistNumber };
    } else {
      throw new Error('User not found');
    }
  } catch (error) {
    console.error('Error retrieving waitlist info:', error);
    throw error;  // Handle the error in the calling code if needed
  }
};

const displayWaitlistInfo = async () => {
  try {
    const { uniqueUrl, waitlistNumber } = await getWaitlistInfo(email, phone);
    
    // Populate the UI with the waitlist info
    console.log('Unique URL:', uniqueUrl);
    console.log('Waitlist Number:', waitlistNumber);

    // You can now insert these values into the HTML or Svelte component
    populateWaitlistInfo = waitlistNumber;
    waitlistLink= uniqueUrl;
  } catch (error) {
    console.error('Error displaying waitlist info:', error);
  }
};

$: phone = formatPhoneNumber(phone);

  function formatPhoneNumber(value: string): string {
    if (!value) return value;

    const inputValue = value.replace(/\D/g, ""); // Remove non-digits
    const phoneNumberLength = inputValue.length;

    if (phoneNumberLength < 4) {
      return `(${inputValue}`;
    } else if (phoneNumberLength < 7) {
      return `(${inputValue.slice(0, 3)}) ${inputValue.slice(3)}`;
    } else {
      return `(${inputValue.slice(0, 3)}) ${inputValue.slice(3, 6)}-${inputValue.slice(6, 10)}`;
    }
  }
</script>
  
  <main class="bg-indigo text-white px-16 pt-16 min-h-[100vh]">
    <!-- Navigation -->
    <header class="flex justify-between flex-col md:flex-row items-center md:justify-end mb-32 md:mb-64">
      <!-- <a href="/about" class="underline text-white self-end md:self-center text-base mb-32 md:mb-0 grow md:grow-0 md:order-2">About</a> -->
      <button on:click={() => handleWaitlistAction("initial")}>
        <img src="../src/assets/logo-white.png" alt="thirdspace logo" class="w-[initial] h-[40px] md:order-1 cursor-pointer">
      </button>
    </header>
  
    <!-- Main Section -->
    <section class="flex flex-col items-center">
      <div class="mb-32 flex flex-col lg:flex-row lg:justify-center items-center w-full gap-[32px] md:gap-[16px]">
        {#if showNewBlock == "initial"} <!-- Right: Waitlist Form -->
        <div class="text-white rounded-lg w-full max-w-lg flex-col">
          <h4 class="text-2xl font-bold mb-8 uppercase hidden md:block">Introducing Thirdspace</h4>
          <h2 class="text-[40px] mb-16 font-bold leading-[49px]">presto chango now you have a friendo</h2>
          
          <form on:submit={handleSubmit} class="flex flex-col text-dark-charcoal">
            <input 
              type="email" 
              placeholder="Email" 
              bind:value={email} 
              class="mb-16 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
            />
            <input 
              type="text"
              id="phone"
              maxlength="14" 
              placeholder="Phone Number" 
              bind:value={phone} 
              class="mb-8 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
            />
            <p class="text-sm mb-16 bold-text text-white">We’ll only use your phone number to send a one-time text with the launch link. No spam, no sharing.</p>
            <button type=submit class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Join waitlist</button>
          </form>
          
          <button on:click={() => handleWaitlistAction("waitlistCheck")} class="underline bold-text text-sm text-white mt-[-8px] block text-center mb-32">Already joined? Check waitlist number.<button/>
        </div>
        {:else if showNewBlock == "waitlist success"}
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left">
          <h2 class="font-bold mb-16">Yay! You're on the waitlist.</h2>
          <p class="mb-16 bold-text text-lg">Skip ahead in line by referring friends. Top 15 get one month premium free.</p>
        
          <div class="flex justify-center items-center mb-16 space-x-2">
            <!-- Input field -->
            <input 
              type="text" 
              placeholder="https://jointthirdspace.com/fakeyfake"
              bind:value={waitlistLink}
              readonly 
              class="unique-url text-xl block w-full px-8 py-8 border-2 border-medium-indigo text-dark-charcoal rounded-md"
            />
            <!-- Copy button (icon) -->
            <button on:click={() => handleWaitlistAction("copyToClipboard")}>
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
          <p class="text-base text-center bold-text">Waitlist Number: {populateWaitlistInfo}</p>
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
            type="text"
            id="phone"
            placeholder="Phone Number" 
            bind:value={phone} 
            class="unique-url mb-16 text-xl block w-full px-8 py-8 border-2 border-medium-indigo rounded-md"
          />
          <button on:click={() => handleWaitlistAction("waitlistCheckSuccess")} class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Check waitlist</button>
          <button on:click={() => handleWaitlistAction("initial")} class="text-base block underline text-white text-left mt-[-8px] bold-text">Join waitlist</button>
        </form>
        {:else if showNewBlock == "waitlist check success"}
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left">
          <h2 class="font-bold mb-16">Yay! You're {populateWaitlistInfo} on the waitlist.</h2>
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
            <button on:click={() => handleWaitlistAction("copyToClipboard")}>
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

  