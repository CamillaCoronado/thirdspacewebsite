<script lang="ts">
  import { firestore } from '$lib/utils/firebaseSetup';
  import { collection, addDoc, query, where, getDocs, getDoc, setDoc, doc, updateDoc, increment } from 'firebase/firestore';
  import { v4 as uuidv4 } from 'uuid';
  let email = '';
  let phone = '';
  let waitlistLink: string = "";
  let populateWaitlistInfo: number;

  let showNewBlock: string = "initial";
  let authError: string = "";
  let uniqueID: string | null = '';

export const handleReferral = async (uniqueID: string) => {
  const referrerRef = doc(firestore, 'waitlist', uniqueID);
  incrementReferralCount(uniqueID);
};


const handleSubmit = async (e: Event) => {
  e.preventDefault();
  
  try {
    const result = await addToWaitlist(email, phone);

    if (result) {
    handleWaitlistAction("waitlistSuccess");
  } else {
    console.error("Failed to add to waitlist");
    return;
  }

    const urlParams = new URLSearchParams(window.location.search);
    uniqueID = urlParams.get('referral');

    if (uniqueID) {
      await handleReferral(uniqueID);
    }

  } catch (error) {
    console.error("Error adding to waitlist:", error);
    const success = false;
  }
};


const getNextWaitlistNumber = async () => {
  try {
    const metaDocRef = doc(firestore, 'meta', 'waitlistTracker');
    const metaDoc = await getDoc(metaDocRef);

    if (metaDoc.exists()) {
      const data = metaDoc.data();
      const currentNumber = data?.currentWaitlistNumber || 0;

      await updateDoc(metaDocRef, {
        currentWaitlistNumber: currentNumber + 1,
      });

      return currentNumber + 1;
    } else {
      await setDoc(metaDocRef, { currentWaitlistNumber: 1 });
      return 1;
    }
  } catch (error) {
    console.error("Error in getNextWaitlistNumber:", error);
    throw error;
  }
};

const recalculateWaitlistPositions = async () => {
  try {
    const waitlistRef = collection(firestore, 'waitlist');
    const querySnapshot = await getDocs(waitlistRef);

    const calculateNewPosition = (originalPosition: number, referralCount: number) => {
      const thresholds = [
        { maxPos: 50, divisor: 1 },
        { maxPos: 20, divisor: 2 },
        { maxPos: 10, divisor: 3 },
        { maxPos: 0, divisor: 5 }
      ];
      const { divisor } = thresholds.find(t => originalPosition > t.maxPos) || { divisor: 1 };
      return Math.max(1, originalPosition - Math.floor(referralCount / divisor));
    };

    const waitlistUsers = querySnapshot.docs.map(doc => ({
      id: doc.id,
      original_position: doc.data().waitlist_number || 0,
      referral_count: doc.data().referral_count || 0,
      new_position: calculateNewPosition(doc.data().waitlist_number || 0, doc.data().referral_count || 0)
    })).sort((a, b) => a.new_position - b.new_position);

    await Promise.all(
  waitlistUsers.map(user => {
    console.log('Updating waitlist number to:', user.new_position);
    return updateDoc(doc(firestore, 'waitlist', user.id), { waitlist_number: user.new_position });
  })
);

  } catch (error) {
    console.error('Error recalculating waitlist positions:', error);
  }
};

export const incrementReferralCount = async (uniqueId: string) => {
  try {
    const waitlistRef = collection(firestore, 'waitlist');
    const q = query(waitlistRef, where('unique_link', '==', `https://jointhirdspace.com/?referral=${uniqueId}`));
    // const q = query(waitlistRef, where('unique_link', '==', `http://localhost:5173/?referral=${uniqueId}`));
    const querySnapshot = await getDocs(q);

    if (!querySnapshot.empty) {
      const docRef = querySnapshot.docs[0].ref;
      await updateDoc(docRef, {
        referral_count: increment(1),
      });
      await recalculateWaitlistPositions();
    }
  } catch (error) {
    console.error('Error updating referral count: ', error);
    throw error;
  }
};

export const getReferralCount = async () => {
  try {
    const waitlistRef = collection(firestore, 'waitlist');
    const urlParams = new URLSearchParams(window.location.search);
    const uniqueID = urlParams.get('referral');

    const q = query(waitlistRef, where('unique_link', '==', `https://jointhirdspace.com/?referral=${uniqueID}`));
    const querySnapshot = await getDocs(q);

    if (!querySnapshot.empty) {
      const userDoc = querySnapshot.docs[0];
      const userData = userDoc.data();
      
      return userData.referral_count;
    } else {
      throw new Error('Document not found');
    }
  } catch (error) {
    console.error('Error getting referral count: ', error);
    throw error;
  }
};

const handleWaitlistAction = async (action: string) => {
  switch (action) {
    case 'copyToClipboard':
      navigator.clipboard.writeText(waitlistLink);
      alert("Link copied to clipboard!");
      break;
    case 'waitlistSuccess':
      await displayWaitlistInfo(); 
      showNewBlock = "waitlist success";
      clearAllTextInputs();
      clearErrors();
      break;
    case 'waitlistCheck':
      showNewBlock = "waitlist check";
      clearAllTextInputs();
      clearErrors();
      break;
    case 'waitlistCheckSuccess':
      const success = await displayWaitlistInfo();

      if (!success) {
          console.error("Failed to display waitlist info, stopping further execution.");
          break;
      }

      showNewBlock = "waitlist check success";
      clearAllTextInputs();
      clearErrors();
      break;
    case 'initial':
      showNewBlock = "initial";
      break;
    default:
      console.error("Unknown action");
  }
};

const generateUniqueLink = (): string => {
  const uniqueId = uuidv4();
  return `https://jointhirdspace.com/?referral=${uniqueId}`;
  // return `http://localhost:5173/?referral=${uniqueId}`;
};

export const addToWaitlist = async (email?: string, phone?: string) => {
  try {
    if (!email && !phone)  {
      handleError("Please provide email or phone number to join the waitlist.")
      return false;
    }

    if (email && !validateEmail(email)) return false;

    const waitlistRef = collection(firestore, 'waitlist');
    
    const uniqueLink = generateUniqueLink();
    const waitlistNumber = await getNextWaitlistNumber();
    const referralCount = 0;

    const data: Partial<{ email: string; phone: string; waitlist_number: number; unique_link: string; referral_count: number; }> = {
      waitlist_number: waitlistNumber,
      unique_link: uniqueLink,
      referral_count: referralCount,
    };
    
    if (email) data.email = email;
    if (phone) data.phone = phone;

    await addDoc(waitlistRef, data);

    return true;
  } catch (error) {
    console.error('Error writing document: ', error);
    throw error;
  }
};

export const getWaitlistInfo = async (email?: string, phone?: string) => {
  try {

    if (!email && !phone) {
      handleError("Either email or phone must be provided.");
      return false;
    }
    const waitlistRef = collection(firestore, 'waitlist');
    const q = email ? query(waitlistRef, where('email', '==', email)) : query(waitlistRef, where('phone', '==', phone));
    const querySnapshot = await getDocs(q);
    
    if (!querySnapshot.empty) {
      const userDoc = querySnapshot.docs[0];
      const userData = userDoc.data();

      const uniqueUrl = userData.unique_link;
      const waitlistNumber = userData.waitlist_number;

      return { uniqueUrl, waitlistNumber };
    } else {
      throw new Error('User not found');
    }
  } catch (error) {
    console.error('Error retrieving waitlist info:', error);
    handleError("Email or phone number not found.") // Handle the error in the calling code if needed
    return false;
  }
};

const displayWaitlistInfo = async (): Promise<boolean> => {
  try {
    const result = await getWaitlistInfo(email, phone);

    // Check if result is false, if so, handle the error
    if (!result) {
      handleError('Error: No waitlist info found');
      return false;
    }

    // Destructure the result since it's no longer false
    const { uniqueUrl, waitlistNumber } = result;

    populateWaitlistInfo = waitlistNumber;
    waitlistLink = uniqueUrl;
  } catch (error) {
    handleError('Error displaying waitlist info');
    return false;
  }

  return true;
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

function clearAllTextInputs(): void {
  phone = '';
  email = '';
}

export function validateEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[a-zA-Z]{2,63}$/;
  const parts = email.split('@');
  const domainParts = parts.length === 2 ? parts[1].split('.') : [];
  const tld = domainParts[domainParts.length - 1]?.toLowerCase();
  const validTLDs = ['com', 'org', 'net', 'edu', 'gov', 'mil', 'io', 'co', 'uk', 'de', 'fr', 'jp', 'au', 'nz'];

  if (
    !emailRegex.test(email) || 
    email.length > 254 || 
    parts.length !== 2 || 
    parts[0].length === 0 || 
    parts[0].length > 64 || 
    domainParts.length < 2 || 
    !validTLDs.includes(tld)
  ) {
    
    handleError("Invalid email format");
    return false;
  }

  clearErrors();
  return true;
}

export function clearErrors(): void {
  authError = '';
}

export function handleError(errorMessage: string): void { 
  const errorElement = document.querySelector('.error-message');
  if (errorElement) {
    errorElement.classList.add('error');
  }
  authError = errorMessage;
}

</script>
  <!-- <button class=" bg-white" on:click={() => clearAllTextInputs()}>button</button> -->
  <main class="bg-indigo text-white px-16 pt-16 min-h-[100vh]">
    <header class="flex justify-between flex-col md:flex-row items-center md:justify-end mb-32 md:mb-64">
      <!-- <a href="/about" class="underline text-white self-end md:self-center text-base mb-32 md:mb-0 grow md:grow-0 md:order-2">About</a> -->
      <button on:click={() => handleWaitlistAction("initial")}>
        <img src="/assets/logo-white.png" alt="thirdspace logo" class="w-[initial] h-[40px] md:order-1 cursor-pointer">
      </button>
    </header>
    <section class="flex flex-col items-center">
      <div class="mb-32 flex flex-col lg:flex-row lg:justify-center items-center w-full gap-[32px] md:gap-[16px]">
        <div class="text-white rounded-lg w-full max-w-lg flex-col transition-all duration-700 ease-in-out opacity-100 h-auto overflow-hidden" 
          class:opacity-100={showNewBlock == 'initial'}
          class:opacity-0={showNewBlock != 'initial'}
          class:h-auto={showNewBlock == 'initial'}
          class:h-0={showNewBlock != 'initial'}>
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
            <p class="text-sm mb-16 bold-text text-white">(Optional) We’ll only use your phone number to send a one-time text with the launch link. No spam, no sharing.</p>
            <button type=submit class="bold-text bg-white text-indigo text-bold btn w-full rounded-full py-12">Join waitlist</button>
            <p class= "error-message">{authError}</p>
          </form>
          
          <button on:click={() => handleWaitlistAction("waitlistCheck")} class="underline bold-text text-sm text-white mt-[-8px] block text-center mb-32">Already joined? Check waitlist number.<button/>
        </div>
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left transition-all duration-700 ease-in-out opacity-100 h-auto overflow-hidden" 
          class:opacity-100={showNewBlock == 'waitlist success'}
          class:opacity-0={showNewBlock != 'waitlist success'}
          class:h-auto={showNewBlock == 'waitlist success'}
          class:h-0={showNewBlock != 'waitlist success'}>
          <h2 class="font-bold mb-16">Yay! You're on the waitlist.</h2>
          <p class="mb-16 bold-text text-lg">Skip ahead in line by referring friends. Top 15 get three months of premium free!</p>
        
          <div class="flex justify-center items-center mb-16 space-x-2">
            <input 
              type="text" 
              placeholder="ERROR"
              bind:value={waitlistLink}
              readonly 
              class="unique-url text-xl block w-full px-8 py-8 border-2 border-medium-indigo text-dark-charcoal rounded-md"
            />
            <button on:click={() => handleWaitlistAction("copyToClipboard")}>
              <img src="/assets/copy-icon.png" alt="Copy" class="w-[32px] h-[34px]" />
            </button>
          </div>
          <div class="flex justify-center gap-[16px] mb-8">
            <a 
              href="https://www.instagram.com/jointhirdspace/"
              target="_blank"
              class="block">
                <img 
                  src="/assets/instagram-logo.png"
                  alt="Instagram"
                  class="w-10 h-10" />
            </a>
            <a 
            href="https://twitter.com/JoinThirdSpace"
            target="_blank"
            class="block">
              <img src="/assets/twitter-logo.png"
                alt="X (Twitter)"
                class="w-10 h-10" />
            </a>
            <a 
              href="https://www.tiktok.com/@jointhirdspace?_t=8q8Z1ft62Tn&_r=1"
              target="_blank"
              class="block">
                <img 
                  src="/assets/tiktok-logo.png" 
                  alt="tiktok" 
                  class="w-10 h-10" />
            </a>
          </div>
        
          <!-- Waitlist Number -->
          <p class="text-base text-center bold-text">Waitlist Number: {populateWaitlistInfo}</p>
        </div>
        <form class="flex flex-col max-w-lg text-dark-charcoal transition-all duration-700 ease-in-out opacity-100 h-auto overflow-hidden" 
          class:opacity-100={showNewBlock == 'waitlist check'}
          class:opacity-0={showNewBlock != 'waitlist check'}
          class:h-auto={showNewBlock == 'waitlist check'}
          class:h-0={showNewBlock != 'waitlist check'}>
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
          <p class= "error-message">{authError}</p>
          <button on:click={() => handleWaitlistAction("initial")} class="text-base block underline text-white text-left mt-[-8px] bold-text">Join waitlist</button>
        </form>
        <div class="bg-indigo w-full max-w-lg text-white rounded-lg text-left transition-all duration-700 ease-in-out opacity-100 h-auto overflow-hidden" 
          class:opacity-100={showNewBlock == 'waitlist check success'}
          class:opacity-0={showNewBlock != 'waitlist check success'}
          class:h-auto={showNewBlock == 'waitlist check success'}
          class:h-0={showNewBlock != 'waitlist check success'}>
          <h2 class="font-bold mb-16">Yay! You're #{populateWaitlistInfo} on the waitlist.</h2>
          <p class="mb-16 bold-text text-lg">Skip ahead in line by referring friends. Top 15 get three months premium free!</p>
        
          <div class="flex justify-center items-center mb-16 space-x-2">
            <input 
              type="text" 
              placeholder="ERROR"
              bind:value={waitlistLink}
              readonly 
              class="text-xl block w-full px-8 py-8 border-2 border-medium-indigo text-dark-charcoal rounded-md"
            />
            <button on:click={() => handleWaitlistAction("copyToClipboard")}>
              <img src="/assets/copy-icon.png" alt="Copy" class="w-[32px] h-[34px]" />
            </button>
          </div>
          <div class="flex justify-center gap-[16px] mb-8">
            <a 
              href="https://www.instagram.com/jointhirdspace/"
              target="_blank"
              class="block">
                <img 
                  src="/assets/instagram-logo.png"
                  alt="Instagram"
                  class="w-10 h-10" />
            </a>
            <a 
            href="https://twitter.com/JoinThirdSpace"
            target="_blank"
            class="block">
              <img src="/assets/twitter-logo.png"
                alt="X (Twitter)"
                class="w-10 h-10" />
            </a>
            <a 
              href="https://www.tiktok.com/@jointhirdspace?_t=8q8Z1ft62Tn&_r=1"
              target="_blank"
              class="block">
                <img 
                  src="/assets/tiktok-logo.png" 
                  alt="tiktok" 
                  class="w-10 h-10" />
            </a>
          </div>
        </div>
        <div class="flex flex-row flex-wrap md:flex-row justify-center items-start mb-16 lg:mb-0">
          <img src="/assets/onboarding-page-1.png" alt="Map your path" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px]">
          <img src="/assets/onboarding-page-2.png" alt="Instant Match" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px]">
          <div class= "grow flex justify-center">
            <img src="/assets/onboarding-page-3.png" alt="Meet and Share" class="mb-16 w-[175px] md:w-[185px] lg:w-[154px] xl:w-[197px] flex-center">
          </div>
        </div>
      </div>
    </section>
  </main>
  <img src="/assets/astronaut.png" alt="Astronaut" class="astronaut overflow-hidden object-contain h-[78px] w-[59px]"
  >

  