---
layout: page
title: Calendrier
description: Évènements divers en rapport avec les archives.
---

<!-- Fenêtre modale -->
<div id="eventModal" class="modal">
  <div class="modal-content">
    <span id="modalClose" class="modal-close">&times;</span>
    <h2 id="modalTitle"></h2>
    <p id="modalTime"></p>
    <p id="modalDescription"></p>
  </div>
</div>


<link href="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.10/main.min.css" rel="stylesheet">
<div id="calendar" style="max-width: 900px; margin: 2rem auto;"></div>

<script src="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.10/index.global.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.10/locales/fr.global.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/ical.js/1.4.0/ical.min.js"></script>

<script>
(async function() {

  const icsUrl = "https://calendar.proton.me/api/calendar/v1/url/euCrRpLMkrVs1gyzq-Ec9Wv6MzZYpXpMCLnGGsOcJLIGXXNVGXf0tUEwqoa9ULqNKodh776gXXXZiBRu9-s-nQ==/calendar.ics?CacheKey=nCcl_qC6YEfpt1-h_Y2M6A%3D%3D&PassphraseKey=PY7nGHxgX08V0Y0CJW77tIcJrGj0EQPFZWX55QJFYKI%3D";

  let text;
  try {
    const response = await fetch(icsUrl);
    text = await response.text();
  } catch (e) {
    console.error("Erreur de chargement ICS :", e);
    return;
  }

  let jcalData, comp;
  try {
    jcalData = ICAL.parse(text);
    comp = new ICAL.Component(jcalData);
  } catch (e) {
    console.error("Erreur de parsing ICS :", e);
    return;
  }

  const vevents = comp.getAllSubcomponents("vevent");

  const events = vevents.map(v => {
    const wrapped = new ICAL.Event(v);

    // Méthode sécurisée : récupère la description complète peu importe la structure
    let description = "";
    try {
      description = v.getFirstPropertyValue("description") || "";
    } catch (e) {
      console.warn("Impossible de lire DESCRIPTION pour l’événement :", wrapped.summary);
    }

    return {
      title: wrapped.summary,
      start: wrapped.startDate.toJSDate(),
      end: wrapped.endDate.toJSDate(),
      allDay: wrapped.startDate.isDate,
      extendedProps: {
        description: description
      }
    };
  });

  const calendar = new FullCalendar.Calendar(document.getElementById("calendar"), {
    locale: "fr",
    firstDay: 1,
    initialView: "dayGridMonth",
    headerToolbar: {
      left: "prev,next today",
      center: "title",
      right: "dayGridMonth"
    },
    events: events,

    eventClick(info) {
      const modal = document.getElementById("eventModal");
        const modalTitle = document.getElementById("modalTitle");
        const modalTime = document.getElementById("modalTime");
        const modalDesc = document.getElementById("modalDescription");

        const start = info.event.start;
        const end = info.event.end;

        function formatDateTime(date) {
            if (!date) return "";
            // Si l’heure est 00:00 et que les minutes sont 0 → événement "toute la journée"
            if (date.getHours() === 0 && date.getMinutes() === 0) {
                return date.toLocaleDateString("fr", { dateStyle: "long" });
            }
            // Sinon, afficher date + heure
            return date.toLocaleString("fr", { dateStyle: "long", timeStyle: "short" });
            }

        // Utilisation
        const startStr = formatDateTime(start);
        const endStr   = formatDateTime(end);

        modalTitle.textContent = info.event.title;
        modalTime.textContent = startStr + " – " + endStr;
        modalDesc.textContent = info.event.extendedProps.description || "";

        modal.style.display = "block";
        }
    });

    calendar.render();

    const modal = document.getElementById("eventModal");
    const closeBtn = document.getElementById("modalClose");

    closeBtn.onclick = function () {
    modal.style.display = "none";
    };

    window.onclick = function (event) {
    if (event.target === modal) {
        modal.style.display = "none";
    }
    };

})();
</script>

Ce calendrier regroupe les différents évènements en liens avec les archives.
