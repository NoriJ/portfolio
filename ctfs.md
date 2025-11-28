# 🏆 Mes CTFs

Ceci est un aperçu dynamique de mes équipes et de leur classement mondial.

<div id="team-393213-info">
  Chargement des données CTFtime...
</div>

<div id="all-teams-ranking">
  </div>

<script>
const TEAM_IDS = [393213, 35520, 287745];
const MAIN_TEAM_ID = 393213;
const API_BASE = "https://ctftime.org/api/v1/teams/";

// Fonction pour récupérer les données d'une seule équipe
async function fetchTeamData(teamId) {
    try {
        const url = `${API_BASE}${teamId}/`;
        const response = await fetch(url);
        
        // Vérifie si la requête a réussi (code 200)
        if (!response.ok) {
            throw new Error(`Erreur HTTP: ${response.status}`);
        }
        
        const data = await response.json();
        return data;

    } catch (error) {
        console.error(`Impossible de récupérer les données pour l'ID ${teamId}:`, error);
        return null;
    }
}

// Fonction principale pour afficher les données
async function displayCTFData() {
    
    // 1. Afficher l'équipe principale
    const mainTeamData = await fetchTeamData(MAIN_TEAM_ID);
    const mainContainer = document.getElementById(`team-${MAIN_TEAM_ID}-info`);
    
    if (mainTeamData && mainContainer) {
        // Formatage du résultat en HTML
        const htmlContent = `
            <h3>${mainTeamData.name}</h3>
            <p><strong>Pays :</strong> ${mainTeamData.country}</p>
            <p><strong>Cote (Rating) :</strong> ${mainTeamData.rating.rating_points.toFixed(2)} pts</p>
            <p><strong>Classement mondial actuel :</strong> #${mainTeamData.rating.current_rating_place}</p>
            <a href="https://ctftime.org/team/${MAIN_TEAM_ID}" target="_blank">Voir le profil sur CTFtime →</a>
        `;
        mainContainer.innerHTML = htmlContent;
    } else if (mainContainer) {
        mainContainer.innerHTML = "<p>Erreur: Données de l'équipe principale non disponibles.</p>";
    }
    
    // 2. (Optionnel) Lister toutes les équipes
    // Vous pouvez étendre cette logique pour récupérer et afficher toutes les équipes ici.
    // ...
}

// Lancement de la fonction une fois que la page est entièrement chargée
window.onload = displayCTFData;
</script>


### 🔹 Catégories préférées
- Web  
- Programmation
- Forensics  
- OSINT

---

### 💡 Notes
- Chaque CTF est une opportunité d’apprendre et de progresser  
