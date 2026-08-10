Ti sei mai chiesto quanto è possibile spremere un modello come kimi k3 per la creazione di un progetto con un singolo prompt ? Riuscirò a creare una landing page per questo prodotto una console che voglio pubblicizzare tramite il mio canale youtube di retro gaming 

Kimi k3 è noto pe rle sue capacità su front end development superando anche fable 5 sui benchmark ( at least is what they said)

# Fase 1 : creazione dei prompt

Dobbiamo creare due prompt uno per vs code ed un altro per la creazione di uno spot pubblicitario da insierire pe ri miei video su youtube ( per 5 secondi in genere un modello come kilng 3.0 pro dovrebbe costare 0,80 centesimi per 5 secondi che dovrebbero bastare anzi li facciamo bastare):

1 orompt : chiederò come al solito di riformulare il prompt a chatgpt sol 5.6 ( high ) 

prompt iniziale :

![[Immagine 2026-07-25 093010.png]]


answer provided: 

 first prompt generation was to overcomplicated to i asked to simplify it this is teh result provided :
 
 Create a modern, responsive landing page for a portable retro gaming console. Use the attached product image in the hero section. Style: - Premium retro-modern design - Dark background - Neon blue and purple accents - Subtle pixel-art details - Clean typography - Smooth, lightweight animations - Mobile-friendly layout Include these sections: 1. Navigation bar 2. Hero section with product image, headline, description and “Buy Now” button 3. Main features 4. Supported retro gaming systems 5. Product gallery 6. Technical specifications 7. Customer review placeholders 8. Price and purchase section 9. FAQ 10. Footer with legal disclaimer Main features: - Portable design - Bright display - Dual analog sticks - Rechargeable battery - Save states - Expandable storage - Headphone support - Multiple emulator compatibility Do not claim that copyrighted games or ROMs are included. Mention that users should only use legally obtained game files. Use: - React - Vite - TypeScript - Tailwind CSS - Lucide icons - Framer Motion Requirements: - Fully responsive - Accessible - Reusable components - Clean and maintainable code - Basic SEO metadata - Mobile navigation menu - Smooth scrolling Use editable placeholders for: - Brand name - Product name - Price - Specifications - Purchase link Generate the complete working project with all files and setup instructions. The project must run with: npm install npm run dev


2 pormpt for ad : 


![[Immagine 2026-07-25 093448.png]]
### Image-to-video prompt

Animate this white retro handheld gaming console in a clean product-commercial style. Keep the console’s exact shape, buttons, colors, proportions, branding, and white background unchanged. The camera slowly pushes in while making a subtle cinematic orbit from left to right. The screen powers on with a soft glow, briefly displays colorful pixel-art graphics, then shows a simple retro game animation. The analog sticks gently tilt, the D-pad presses once, and the colored A, B, X, and Y buttons illuminate sequentially. Add realistic reflections on the screen, soft studio shadows, and subtle plastic highlights. Smooth, controlled motion, premium product advertising, sharp focus, realistic materials, 5-second seamless animation, no hands, no extra objects.

**Negative prompt:**  
Do not deform the console, change the button layout, alter the text, add new controls, rotate it unnaturally, make it float excessively, introduce hands or people, distort the screen, blur the product, or change the background.

### Basic specifications

- **Display:** 3.5-inch IPS, 640 × 480, 4:3
- **Processor:** Rockchip RK3326, quad-core Cortex-A35
- **Graphics:** Mali-G31 MP2
- **Memory:** 1 GB DDR3L RAM
- **Storage:** two microSD slots; this listing appears to offer 64 GB and 128 GB versions
- **Controls:** D-pad, ABXY buttons, two analogue sticks and four rear shoulder buttons
- **System:** Linux, normally ArkOS
- **Connections:** two USB-C ports and a 3.5 mm headphone socket
- **Wireless:** normally no integrated Wi-Fi or Bluetooth; Wi-Fi requires a USB adapter
- **Weight:** approximately 180–187 g

### What it can emulate

It is excellent for **NES, SNES, Mega Drive, Game Boy, Game Boy Color, Game Boy Advance, arcade games and PlayStation 1**. Nintendo 64, Dreamcast, Nintendo DS and PSP games have mixed performance: simpler titles may work, but demanding games can stutter or require configuration. It is not suitable for GameCube, PS2 or newer systems.pecification.

# let's  dive in :

i will ask to vs code to send the request to kimi k3 trough open router in a fresh new environment: 


i've just copied the pompt previously genrated in a prompt.txt for kimi to read


![[Immagine 2026-07-25 100247.png]]


Now let's ask codex to make the requesto trough open router and then sendind the prompt provided :

first i need a fresh openrouter key ![[Immagine 2026-07-25 100850.png]]

ok now let's send the prompt and let's see what happens and how much it takes : ![[Immagine 2026-07-25 101043.png]


ok we ran into a problem:


![[Immagine 2026-07-25 102909.png]]


the answer say that the request i completed but the balance allowed only 2600 output token let's see how much token the request consumed: 

![[Immagine 2026-07-25 103214.png]]


ok the request burned all the credit ( 2 $ budget ) we're kind of screwed, i need to find another solution let's see if we can find some ways to make this simple request for free: 

on kimi k3 official website there is  a waitlist to upgrade account ( as to be expected) but the price for even the basic account is obviously out fo budget , let's try to make a simple request for code in html and then pass it to vscode trough chatgpt sol ( for which i have plus sub ) to implement ( in this way the two model kinda work togheter):

ok it worked , i sented this prompt : 
![[Immagine 2026-07-25 104733.png]]

and the model came out with that :
![[Immagine 2026-07-25 110039.png]]

the output tokens provided was about : 
![[Immagine 2026-07-25 110109.png]]
roughly **$0,195**  in token value , the result looks a little bit basic , i think we can make it better just asking to improve; the model knew or searched all the specs of the product ( this model works kinda weel for promoting chinese model console ) and now let's heat up some gpus in china with another request

![[Immagine 2026-07-25 111905.png]]

the model hitted the free limit here : ![[limit hit.png]]
( the request is too high and the switche to k2.6 instant but produced some output let's see ho much tokens we've got ) same output of the same quality as to be expected  for a single  html page let's see how the points ( GPT 5.6 integration points as kimi said) that the mdoel provided for gpt 5,6. 

we will be using the first output and fix some design error as some layout mistakes : 


![[layoutmistakes.png]]


lets' be honest it looks cool : 

![[let's be honest it looks cool.png]]

setting up payment work flow :

![[Pasted image 20260725154456.png]]

Now we need to figure out a way to make an ad considered teh fact that we nee to use a free model considering the fact that ran out of money with the first request.

I've added a cool feature through a telegram bot that send me orders to process with custome rinfo ( i think i will write something later about that).
![[IMG_0154.png]]
Now i need to fix some overlay issues on mobile and i think we're ready to move forward with the ad campaign 

![[IMG_0155 1.png]]

# Ad campaign: 

i have a youtube channel where i upload retro games gameplays, we will make a simple ad with a free model upload on the channel then  (~~229~~) . 

![[Immagine 2026-07-27 110512.png]]

i tried on veo 3.1 to see if i had any credits left from previous project but nope so i switched to capcut. The plan is to make a normal video upload on the channel with the ad and then link as a video in the short so the alghoritms doesn't being messed ( or this is in theory what i belive will happen even i dont' actually know how this specific method works).

![[Immagine 2026-07-27 111847.png]]

Capcut gives actualluy a lot more of free credit to generate video than i expected, so i thing we have enough credits to do multiple genrations. When a user see the story he or she sees only a normal story ( i cannot insert product selling because my channel isn't monetized already so we need ) and he or she sees a correlated video that i uploaded with the title : 🛒 shop now ... or something like that the user click sees the ad and then in the description i will provide link to the product page.

![[Immagine 2026-07-27 111040.png]]

I also provided the same reference image used before with a twist, a spicy one actually ( the prompts i copied before on this article provided some strange results so i made some tweaks)

![[A709B5D6-D060-4C25-9111-39AD2D0B58F2 1.png]]


this was made with nano banana ( first i made the girl with normal clothes then i asked chatgpt image model to make she look like this ).


![[IMG_0197.png]]

Now we have three videos : 
https://youtu.be/AZoRdkSwNf8
https://youtu.be/rqFyqHyuBBQ
https://youtu.be/fB7UoBMKwCA

Project is done total cost : 2 dollar ( that actually went lost) and free credits from free trial subscription plans, i'm actually quite happy and i will upload updates soon.