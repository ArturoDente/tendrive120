# tendrive120
a 10 line - 120 colums driving game for c64, written in basic for homeputerium 10 line contest, 2020 edition
TENDRIVE PUR120 C64 BASIC GAME - By Arturo Dente

*HISTORY*

a real game needs a history, so here is mine: you are a coder driving his f1 car to get his award for the first place in a 10 line code competition, that will be kept 12 miles from home, near the big-code-mountain. 

During the game you have: 
1) a mountain growing on the horizon, the more you get near, the more big it is. 
2) the miles traveled (positioned on the center / left)
3) two multicolor sprites for your car and the enemy one (there are always enemies when you have a competition!)
4) centrifugal force while steering on a curve
5) the street
6) the damage: damage is showed by covering the word "tendrive" whith # .. when the word is gone, you have gameover.

If you get 12 miles, you arrive, and a small cup will be given to you (no more room to make celebrations, sorry).
The highest score is showed in the first screen of the game.

-----------------------------------------------------------

*THE CODE*

	*General thoughts*
Some treat and triks :
	- DATA are scattered everywhere there's a space to do it.
	- Sprites are defined using only the last lines of the memory dedicated to, in order to avoid to DATA a lot of bytes. In reality, I had to zero-fill the empty area because at real hardware start it's covered with random data, not zero. Then, multicolor gives the sprite bigger itself.
	- I had to manually call garbage collector via z=fR(0), because the looping reinitialization of the street string gave soon a string area overflowing other ram sections.
	- Street map is contained in p$="2021202221", where the couples of number stay for "do n1 times the n2 kind of piece of route".  So we have 2 straightaways followed by 2 left courves (or right? I don't remember), etc etc. All the string is processed in order to populate an array of real directives about the route. See code comment for an explanation.
	- When possible, I used ON ... instead of IF in order to obtain a line not logically breaked by the IF lacking of ELSE


	*Initialization parts*
LINE 0 (and 9)
	- Variable s$="  " serves to have a way to print n full spaces wheen needed (no, SPC(n) was not ok, I needed full spaces, generally with {reverse on} before).
	- Variable p$="2021202221". See up.
	- Instructions i=0:fOt=1to9stE2:s$=s$+s$:a$=mI(p$,t,2):fOs=1tovA(leF(a$,1)):p(i)=vA(rI(a$,1)):i=i+1:nE:nE:gO9
		I put in this instructions a filling of s$ of much spaces, the 2 original were not enough. In the meantime, I populate the array p() with the pieces of route. Notice that p$ gives 10 total pieces of route, so DIM not needed.
		In the end we have a jump to line 9 that's the initial screen, with some other initializations. The more important are: v=53248 for sprites, q=56320 for Joystick port #2, o=65520 to call with SYS and some pokes in order to obtain a "print at".

LINE 1
	-Function definition: dEfnz(u)=-u*(u<2)+(u=2). I made this because p$ could not contain negative numbers (remember, i parse p$ with couples, a "-" sign would break this) , so basically this functions returns its argument when it is 0 or 1, and gives -1 when the argument is 2.
	-Instructions c=12288:fOs=ctos+33:pOs,0:nE:fOt=stot+28:rEs:pOt,s:nE
		These instrucions build the sprite. Zero fill from 12288 to 12288+33, then read data for the rest of the sprite. Other stuff is sprite initialization related, or variables initializated.

LINE 2
	Here we have some initialization (one important is c$="{cm +}{cm +}", that's the blocks used for the bounds of the street and the mountain) and the code to fill the screen with stars when the game starts. Here we use for the first time the variable s$ to get some spaces filled.   fOt=1to160:b$="{reverse on}{black}"+leF(s$,rN(t)*9)+".{reverse off}":?b$;

LINE 3
	other variables, expecially c$(0)="{white}":c$(1)="{gray}" that are used to alternate street bounds with colors white and gray, and k$="{home}{down}{down}{down}{down}{down}{down}{down}{white}{reverse on}"+leF(s$,39)+"{right}{up}" that is used to display a message in the middle-left of the screen (for example the miles). Note that it ends with right+up after 39th char,so it is positioned to the start of the string just printed on video.

LINE 4
	Code: 4z=fnz(p(p)):fOi=1to2:fOt=0to5:g=z*i*t/2:d=g+t/2:w$="{reverse on}{green}"+leF(s$,8+d)+c$(-b)+c$+"{reverse off}"+leF(s$,20+g-t)+c$(-b)+c$:pO780,0:dA60
	Explanation: we start here the main cycle, printing the street. Variable z uses the defined function already expained, in order to obtain a zero/positive/negative factor for a delta. Printing has two levels of degree expecially when painting curves. The two variables FORs are intended to do this. First, i=1to2 (two values), and - inside this cycle - t=0to5 (six values) in order to have the 6 stripes of street rendered two by two. Variable w$ will contain the row to print. We calculate  g=z*i*t/2 and d=g+t/2 in order to have the spaces before the street stripe starts (the green leF(s$,8+d) spaces), followed by the C$ to make bounds (alternatively coloured by c$(-b), where b is not b every NEXT) , followed by the gray part (the street for the car) , leF(s$,20+g-t) spaces long  , followed by c$ to close the bound. 
	
	pO780,0 is part of the "print at" routine.

LINE 5
	Code: 5pO781,24-t:pO782,0:sYo:?w$"{reverse on}{green}"leF(s$,44-len(w$));:pOv+2,150+10*p(p):iff=0tH?k$sP9)"go":dA63,255,252,62,125,188,62,170

	Explanation: pO781,24-t is the variable part of the "print at", it gives the y of the row to be printed. SYS o is called before ?w$"{reverse on}{green}"leF(s$,44-len(w$)); Note that we are printing w$ followed by green filling for the remaining part of the row. I could'nt put this part directly in w$ for room reasons.

LINE 6
	Code: 6pO1024+ht,35:j=pE(q):x=x+4*(j=uorj=123)*(1+2*(j=u))-z:ht=ht-b*(pE(v+30)>0orx<100orx>224):on2+(ht=0)gO9:iff=13tH?"{clear}A":eN
	
	Explanation: 	pO1024+ht,35 updates the hits status on screen.
			j=pE(q):x=x+4*(j=uorj=123)*(1+2*(j=u))-z : updates car position. Note the -z at the end, this is the centrifugal force!
			ht=ht-b*(pE(v+30)>0orx<100orx>224): this updates the hits variables, by sprite vs sprite collision or x boundaries. I multiply for b, the boolean used for alternate colors of street bounds, in order to keep the difficult a bit lower. You are gracied once yes, once no.
			on2+(ht=0)gO9: this is the "if" to get the end of the game for excessive damage.
			iff=13tH?"{clear}A":eN : this is the if to get the end of the game for arriving to destination. Note the big celebration party with the "cup" followed by the end.. "{clear}A":eN

LINE 7
	Code: 7ifn>mtHn=l:f=f+1:?k$+stR(f)+"{black}"+leF(s$,37):r$="{brown}":fOzz=1tof:r$=r$+c$:pO780,0:pO781,18+zz-f:pO782,20-zz:sYo:?r$;:nE

	Explanation: we are still in the double FOR cycle of line 4. This line is all conditioned to the ifn>m, that stays for the end of a "lap", that is all the p() array has been consumed and printed. So we update the miles (used as points,variable f) , then we update on screen the miles with ?k$+stR(f)+"{black}"+leF(s$,37) (that fill the remaining space with black spaces), and finally we print the right section of the mountain with the code:
	r$="{brown}":fOzz=1tof:r$=r$+c$:pO780,0:pO781,18+zz-f:pO782,20-zz:sYo:?r$;:nE
	r$ is filled with c$ in proportion of the miles traveled (variable f), and because a line has the lentght of the line up plus one c$, we can print from top to bottom incrementing the length.

 LINE 8

	Code: 8pOv,x:p=p+1:p=-p*(p<11):b=nOb:nE:b=nOb:nE:n=n+10:pOv+21,3:pOv+3,n:h=-f*(f>h)-h*(f<=h):z=fR(0):gO4:dA188,60,0,60
 
	Explanation: here we are closing the double FOR cycle, incrementing some variables and checking the highscore h. We call then garbage collector and restart the cycle with GOTO 4


