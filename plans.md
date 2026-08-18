Implements a single plans.html file, show forms as modal popups

Use a white background for the page (light theme).

---
Mostrare il logo trustable.png centrato all'inizio

Pricing UI – struttura completa a card e tab

mostrare un tab to switch "entry plans", "professional plans", "custom plans"

Mostrare uno switch "Monthly / Yearly" sopra le card.
Quando è selezionato Yearly, il prezzo annuale è pari a `monthly * 10`.
Lo switch non si applica al piano Hobby (Free) né a Custom Solutions.

⸻

Tab 1 — Entry Plans

Hobby
	•	Price: Free
	•	AI: Self-hosted
	•	Only local applications (no publishing)
	•	CTA: Configure Servers (secondary button)

⸻

Freelance
	•	Price: $20/month
	•	AI: Included – 20M tokens/day
	•	Server: One app on Nuvolaris.Dev
	•	CTA: Free Trial 30 days (primary button, evidenziato)

⸻

Tab 2 — Professional Plans

Tre card affiancate con progressione chiara (da sinistra a destra più potenza + prezzo).

Mostrare il logo nuvolaris.png in basso a sinistra

Startup
	•	Subtitle: Dedicated VM, unlimited apps
	•	AI: 50M tokens/day
	•	Server: 1 node
	•	Price: $50/month
	•	Extras: Professional Support
	•	CTA: Buy This

⸻

Scaleup
	•	Subtitle: Cluster deployment
	•	AI: 100M tokens/day
	•	Server: Cluster (1 master, 2 workers)
	•	Price: $200/month
	•	CTA: Buy This
	•	Badge: Most Popular

⸻

Enterprise
	•	Subtitle: High Availability cluster
	•	AI: 200M tokens/day
	•	Server: HA Cluster (3 masters, 3 workers)
	•	Price: $500/month
	•	CTA: Buy This

⸻

Tab 3 — Custom Solutions

Mostrare il logo bestia.png a sinistra

BestIA Custom Solutions
	•	Style: card full-width (più grande delle altre)
	•	Features:
	•	Site License for Trustable
	•	Private AI with on-premises GPU
	•	Nuvolaris Enterprise License
	•	Monitoring and Backup
	•	Proactive Enterprise Support
	•	CTA: Contact Sales (button prominente)

⸻

Comportamento UI
	•	Card con:
	•	bordo sottile + glow su hover
	•	CTA più evidente nel piano consigliato
	•	Progressione visiva:
	•	Hobby → minimal
	•	Starter → evidenziato
	•	Professional → scala crescente
	•	Enterprise → dominante, full-width

⸻

Note UX (importanti per conversione)
	•	“Starter” evidenziato per onboarding
	•	“Professional Medium” come anchor pricing
	•	Enterprise separato per evitare overload decisionale
	•	Token/day sempre visibile → metrica chiave

---

Se l'utente sceglie Hobby
il wizard deve chiedere host e porta del server.
Deve mostrare che l'URL sarà `http://<host>:<port>/v1`.
Deve mostrare chiaramente che il server deve essere raggiungibile in HTTP, senza autenticazione.
Deve esserci un bottone per testare l'accesso.

Se l'utente sceglie Starter o Pfoessiona

il wizard chiedere email password, confirm password, br, cognome, email e le altre informazioni necessarie per attivare l'account.

poi chiede la carta di credito


se l'utente ha scelto starter deve dire
 nothing will be charged today, cancell any time

we will warn you by email before charging


Se l'utente sceglie enteprise una form di richiesta contatto e poi we will be in touch with you asap


usa delle icone e delle decorazioni per i vari piani