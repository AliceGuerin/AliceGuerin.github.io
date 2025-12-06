---
layout: page
title: Calendrier
description: Évènements divers en rapport avec les archives.
---

<div id="calendar" style="max-width: 1200px"></div>

<!-- Fenêtre modale -->
<div id="eventModal" class="modal" style="display:none;">
  <div class="modal-content">
    <span id="modalClose" class="modal-close">&times;</span>
    <h2 id="modalTitle"></h2>
    <p id="modalTime"></p>
    <p id="modalDescription"></p>
  </div>
</div>

<script src='https://cdn.jsdelivr.net/npm/fullcalendar@6.1.19/index.global.min.js'></script>
<script src='https://cdn.jsdelivr.net/npm/@fullcalendar/core@6.1.15/locales/fr.global.min.js'></script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/ical.js/1.4.0/ical.min.js"></script>

<script>
(async function() {

  // Fonction pour charger et parser le fichier ICS
  async function loadICal(url) {
    const response = await fetch(url);
    if (!response.ok) throw new Error("Impossible de charger l’ICS : " + response.status);
    const text = await response.text();
    return ICAL.parse(text); // retourne l'objet jCal
  }

  let jcalData, comp, vevents;
  try {
    jcalData = await loadICal("/Archives-2025-12-06.ics"); // chemin ICS dans le repo
    comp = new ICAL.Component(jcalData);
    vevents = comp.getAllSubcomponents("vevent");
  } catch (e) {
    console.error("Erreur de chargement ou parsing ICS :", e);
    return;
  }

  // Conversion des événements en format FullCalendar
  const events = vevents.map(v => {
    const wrapped = new ICAL.Event(v);

    let description = "";
    try {
      description = v.getFirstPropertyValue("description") || "";
    } catch(e) {
      console.warn("Impossible de lire DESCRIPTION pour l’événement :", wrapped.summary);
    }

    return {
      title: wrapped.summary,
      start: wrapped.startDate.toJSDate(),
      end: wrapped.endDate.toJSDate(),
      allDay: wrapped.startDate.isDate,
      extendedProps: { description }
    };
  });

  // Initialisation du calendrier
  const calendar = new FullCalendar.Calendar(document.getElementById("calendar"), {
    locale: "fr",
    firstDay: 1,
    initialView: "dayGridMonth",
    headerToolbar: {
      left: "prev,next today",
      center: "title",
      right: "timeGridWeek,dayGridMonth,multiMonthYear"
    },
    events: events,

    eventClick(info) {
      const modal = document.getElementById("eventModal");
      const modalTitle = document.getElementById("modalTitle");
      const modalTime = document.getElementById("modalTime");
      const modalDesc = document.getElementById("modalDescription");

      const start = info.event.start;
      const end = info.event.end;

      // Fonction pour afficher date ou date+heure selon allDay
      function formatDateTime(date) {
        if (!date) return "";
        if (date.getHours() === 0 && date.getMinutes() === 0) {
          return date.toLocaleDateString("fr", { dateStyle: "long" });
        }
        return date.toLocaleString("fr", { dateStyle: "long", timeStyle: "short" });
      }

      const startStr = formatDateTime(start);
      const endStr   = formatDateTime(end);

      modalTitle.textContent = info.event.title;
      modalTime.textContent = startStr + " – " + endStr;
      modalDesc.textContent = info.event.extendedProps.description || "";

      modal.style.display = "block";
    }
  });

  calendar.render();

  // Gestion de la fermeture de la modale
  const modal = document.getElementById("eventModal");
  const closeBtn = document.getElementById("modalClose");

  closeBtn.onclick = function () { modal.style.display = "none"; };
  window.onclick = function (event) { if (event.target === modal) modal.style.display = "none"; };

})();
</script>
