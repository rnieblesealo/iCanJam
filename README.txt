 _  ____                _                 
(_)/ ___|__ _ _ __     | | __ _ _ __ ___  
| | |   / _` | '_ \ _  | |/ _` | '_ ` _ \ 
| | |__| (_| | | | | |_| | (_| | | | | | |
|_|\____\__,_|_| |_|\___/ \__,_|_| |_| |_|
                                          
     .-. __ _ .-.
     |  `  / \  |
     /     '.()--\
    |         '._/
   _| O   _   O |_
   =\    '-'    /=
     '-._____.-'
     /`/\___/\`\
    /\/o     o\/\
   (_|         |_)
     |____,____|
     (____|____)

i like to jam with my friends on ucf's garages, but because of event parking, 
some garages not having power ( for amps, etc. ), and some being restricted,
it can be really hard to plan when to do this

this app fixes that by letting you check which garage you are able to jam in when

WHEN CAN U JAM?

  * garage/spot has power ( on at least one outlet )
      this has to be empirical
      users will have to report whether there is power or is not
      maybe display this as a score?
  * garage/spot is not used for event parking THAT DAY
      webscrape ucf gameday
      addition financial arena too
  * time is after 5pm ( work day is over, less likely to form trouble )

LOITERING DISCLAIMER: 

**UCF-4.006 Trespass and Loitering** forbids loitering in garages

jamming counts as loitering, so it is a clear VIOLATION OF THIS POLICY 
though the enforcement of this is quite lax ( LOTS of bands practice at ucf ),
it's still important to bear in mind and as such users should be EXPLICITLY WARNED

usage of the app BY ITSELF should NOT constitute as a violation, however

( do more research on this! don't get in trouble :) )

==========================================================================================
EMPIRICAL RESEARCH
==========================================================================================

to know if event parking will be in action, we need 3 things:
  a) a list of all games/events taking place at UCF stadiums/complexes
  b) a list of all games/events taking place at the Addition Financial Arena
  c) a list of all parking locations that are used for EVENT PARKING

=== OBTAINING EVENT DATA ( UCFKNIGHTS.COM ) ==============================================

after doing some webscraping of ucfknights.com, i found that THIS URL:

https://ucfknights.com/website-api/schedule-events?filter%5Bupcoming%5D=true&filter%5Bhide_from_all_sports_schedule%5D=false&sort=datetime&include=conference.image,opponent.officialLogo,opponent.customLogo,opponentLogo,schedule.sport,scheduleEventLinks.icon,scheduleEventResult,secondOpponent.officialLogo,secondOpponent.customLogo,secondOpponentLogo,postEventArticle,preEventArticle,presentedBy&per_page=50&page=1

returns info about upcoming games ( ucfknights.com )
	we can filter by games taking place in ORLANDO, FLA,
  then extract the datetime in order to know when parking will be in effect 

=== OBTAINING EVENT DATA ( ADDITION FINANCIAL ) ==========================================

as for addition arena events,
its page's fetched html has the event data prebaked in it: 

https://www.boxofficeticketsales.com/venues/addition-financial-arena?keyword=Addition%20Financial%20Arena&utm_source=bing&utm_medium=cpc&utm_campaign=Venues%3ESS%3ECombined%3ENat%3EHead%20Keyword%3EExact&utm_term=Addition%20Financial%20Arena&utm_content=Venues%3ESS%3ECombined%3ENat%3ETier%201%3EAddition%20Financial%20Arena%3E659%3EOrlando%3EFL%3EEDT%3E534%3EMulti-Purpose%20Venue

the variable "esRequest" contains all event data in prebaked json 
looks like we'll need to extract this object and then get our games data from there

NOTE: the api response time is very slow for this, seemingly bc addition server is slow
  after breaking down the urls via URLSearchParams and cutting some api call fat,
  the response is still slow, meaning this might be outside my control
  
  investigate more if we wish for this to be faster!

=== MARSHALING THE DATA IN A FORMAT THAT'S USEFUL ========================================

THE END GOAL IS TO HAVE A JSON OBJECT LISTING ALL EVENTS THAT TRIGGER UCF EVENT PARKING

** PROCESS SO FAR **

	1. fetch ucf gameday games

	2. fetch financial arena events

  3. extract EVENT NAME, EVENT DATE and EVENT TIME

  4. fetch & extract PARKING LOCATIONS
      openstreetmap to get all parking garages round ucf ( check chatgpt )
      addition financial to get parking garages specific for events

      ( how will we reconcile both datasets? 
        addition financial event parking lots should be a 
        SUBSET of OPENSTREETMAP data... )

  5. if THERE IS ANYTHING TAKING PLACE TODAY,
      ALL PARKING LOCATIONS FROM UCF FI ARE BLOCKED

		  possibly add "filter by needs/instrument"
        percussionists do NOT need outlets
        maybe have a switch: need outlet?

			that modifies the operation:

			for GARAGE in ALL_GARAGES:
				if EVENT THAT DAY & REQUIRE OUTLET & GARAGE IN (EVENT PARKING):
					GARAGE.USABLE = FALSE

** EMERGING PROBLEM **

  games fetched from ucfknights.com that take place in addition arena will be duplicated
  in events fetched from addition arena, likely with slightly different name

  HOW DO WE RECONCILE DUPES INTO ONE OBJECT?

==========================================================================================
TECH STACK
==========================================================================================

* LEAFLET for map + custom data for locations
    rationale: decent AND open source! f***k proprietary bs

* VITE + REACT + TS ( FRONTEND )
    rationale: only need simple SPA; also very comfortable using this to build cool ui 
               mobile-first design is easy and can be react native'd 
               ( though ive never actually done before... )

* DIGITALOCEAN hosting
    rationale: this is lowkey illegal under ucf policy,
               so for peace of mind i want everything running on my own count 

* NAMECHEAP DOMAIN
    rationale: namecheap has a pling.org charm to it which makes me like it 
               also, the names are cheap!!! #win

* POSTGRES DB
    rationale: only DB i've really written before and is quite good/standard

* GOLANG ( BACKEND ) 
    rationale: good way to learn go for my job, also it's a great language for this!
