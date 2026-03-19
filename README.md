# Documentation Complète de la Conception de DeepTeam
## Analyse Approfondie du Framework de Red Teaming pour LLM

---

## Table des Matières

1. [Vue d'Ensemble](#vue-densemble)
2. [Architecture Générale](#architecture-générale)
3. [Composants Principaux](#composants-principaux)
4. [Système de Vulnérabilités](#système-de-vulnérabilités)
5. [Système d'Attaques](#système-dattaques)
6. [Système de Métriques et d'Évaluation](#système-de-métriques-et-dévaluation)
7. [Frameworks de Sécurité](#frameworks-de-sécurité)
8. [Système de Garde-fous (Guardrails)](#système-de-garde-fous-guardrails)
9. [Patterns de Conception](#patterns-de-conception)
10. [Workflow d'Exécution](#workflow-dexécution)
11. [Extensibilité et Customisation](#extensibilité-et-customisation)

---

## Vue d'Ensemble

### Qu'est-ce que DeepTeam?

**DeepTeam** est un framework open-source de **red teaming** conçu pour:
- **Tester la sécurité** des systèmes LLM (Large Language Models) en production
- **Identifier les vulnérabilités** avant qu'elles ne deviennent des risques réels
- **Simuler des attaques adverses** basées sur les dernières techniques de recherche
- **Générer des rapports d'évaluation des risques** détaillés
- **Deployer des garde-fous** pour protéger les systèmes en production

### Objectifs Principaux

1. **Simulation d'Attaques Adverses** : Générer des attaques sophistiquées (jailbreaking, prompt injection, etc.)
2. **Évaluation des Vulnérabilités** : Évaluer 40+ types de vulnérabilités (biais, PII, hallucination, etc.)
3. **Rapports de Risques** : Produire des évaluations structurées conformes aux standards (OWASP, NIST, MITRE)
4. **Protection en Production** : Deployer des garde-fous pour bloquer les attaques détectées
5. **Extensibilité** : Permettre l'ajout de nouvelles vulnérabilités, attaques et métriques

---

## Architecture Générale

### Architecture de Haut Niveau

```
┌─────────────────────────────────────────────────────────────┐
│                      COUCHE UTILISATEUR                      │
│  ┌──────────────┐        ┌──────────────┐                   │
│  │ CLI (main.py)│        │ Programmatic │                   │
│  │              │        │ API          │                   │
│  └──────────────┘        └──────────────┘                   │
└──────────────┬────────────────────────────────────────────┘
               │
       ┌───────┴────────┬──────────────┐
       │                │              │
   ┌───────────┐    ┌────────────┐  ┌──────────┐
   │Configuration    │ModelCallback  │Metadata │
   └───────────┘    └────────────┘  └──────────┘
       │                │              │
       └────────┬───────┴──────────────┘
                │
   ┌────────────▼──────────────────────────────────┐
   │ COUCHE D'ORCHESTRATION: RedTeamer             │
   │  • Gestion du workflow de red teaming         │
   │  • Coordination des composants                │
   │  • Mode async/sync                            │
   │  • Gestion de la concurrence                  │
   └────────────┬──────────────────────────────────┘
                │
   ┌────────────┴────────────────────────────────┐
   │ COUCHE DE COMPOSITION: SimulateurAttaques   │
   │  • Génération d'attaques de base            │
   │  • Amélioration via techniques d'attaque    │
   │  • Single-turn et multi-turn                │
   └────────────┬────────────────────────────────┘
                │
   ┌────────────┴──────────────────────────────────────────┐
   │ COMPOSANTS MÉTIER (disponibles en parallèle)          │
   │                                                       │
   │  ┌──────────────────┐  ┌──────────────────┐          │
   │  │ Vulnérabilités   │  │ Attaques         │          │
   │  │ • 40+ types      │  │ • Single-turn    │          │
   │  │ • Avec métriques │  │ • Multi-turn     │          │
   │  │                  │  │ • Pondérées      │          │
   │  └──────────────────┘  └──────────────────┘          │
   │                                                       │
   │  ┌──────────────────┐  ┌──────────────────┐          │
   │  │ Frameworks       │  │ Guardrails       │          │
   │  │ • OWASP Top 10   │  │ • Input Guards   │          │
   │  │ • NIST           │  │ • Output Guards  │          │
   │  │ • MITRE          │  │ • Safety Levels  │          │
   │  └──────────────────┘  └──────────────────┘          │
   │                                                       │
   │  ┌──────────────────┐                                │
   │  │ Métriques        │                                │
   │  │ • 40+ métriques  │                                │
   │  │ • Scoring LLM    │                                │
   │  │ • Résultats pass/ fail                           │
   │  └──────────────────┘                                │
   └────────────┬──────────────────────────────────────────┘
                │
   ┌────────────▼────────────────────────────────┐
   │ COUCHE DE RÉSULTATS: RiskAssessment         │
   │  • Agrégation des résultats                 │
   │  • Statistiques de risques                  │
   │  • Export JSON/DataFrame                    │
   │  • Upload vers plateforme (optionnel)       │
   └─────────────────────────────────────────────┘

   ┌─────────────────────────────────────────────┐
   │ COUCHE PRODUCTION (Post Red-Teaming)        │
   │  □ Deployer les guardrails                  │
   │  □ Monitorer les patterns d'attaque         │
   │  □ Itérer sur les défenses                  │
   └─────────────────────────────────────────────┘
```

### Flux de Données Principal

```
Configuration/Appel API
        ↓
    RedTeamer.__init__()
        ↓
    red_team(target_model, vulnerabilities/framework, attacks)
        ↓
    [Async/Sync] AttackSimulator.simulate()
        ↓
    Generate baseline attacks + Apply attack techniques (weighted sampling)
        ↓
    Exécution contre le modèle cible (via model_callback)
        ↓
    Évaluation des résultats (via métriques)
        ↓
    Agrégation des résultats
        ↓
    RiskAssessment (résultats + statistiques)
        ↓
    Output (Console, DataFrame, JSON)
```

---

## Composants Principaux

### 1. RedTeamer - Orchestrateur Central

**Fichier** : `red_teamer/red_teamer.py`

#### Responsabilités Clés

```python
class RedTeamer:
    """
    Orchestrateur principal du red teaming.
    Gère l'ensemble du workflow de test de sécurité.
    """
    
    def __init__(
        self,
        simulator_model: Union[str, DeepEvalBaseLLM] = "gpt-3.5-turbo",
        evaluation_model: Union[str, DeepEvalBaseLLM] = "gpt-4o",
        target_purpose: Optional[str] = "",
        async_mode: bool = True,
        max_concurrent: int = 10,
    ):
        # Initialisation des deux modèles LLM
        self.simulator_model      # Génère les attaques
        self.evaluation_model     # Évalue les résultats
        
        # Configuration d'exécution
        self.async_mode           # Mode asynchrone pour parallélisation
        self.max_concurrent       # Nombre de requêtes parallèles
        
        # Composant clé
        self.attack_simulator     # Simulateur d'attaques
```

#### Responsabilités

1. **Initialisation des modèles** via Factory pattern
2. **Gestion du workflow** (sync/async)
3. **Coordination de la simulation** via AttackSimulator
4. **Exécution des tests** contre le modèle cible
5. **Évaluation des résultats** via les métriques
6. **Génération des rapports** (RiskAssessment)
7. **Upload optionnel** vers la plateforme Confident

#### Méthodes Principales

```python
def red_team(
    self,
    model_callback: Union[Callable, str, DeepEvalBaseLLM],
    vulnerabilities: Optional[List[BaseVulnerability]] = None,
    attacks: Optional[List[BaseAttack]] = None,
    framework: Optional[AISafetyFramework] = None,
    attacks_per_vulnerability_type: int = 1,
    ignore_errors: bool = True,
    reuse_simulated_test_cases: bool = False,
    metadata: Optional[dict] = None,
) → RiskAssessment:
    """
    Point d'entrée principal pour le red teaming.
    
    Paramètres:
    - model_callback: Fonction/modèle cible à tester
    - vulnerabilities: Liste de vulnérabilités à évaluer
    - attacks: Liste d'attaques à utiliser
    - framework: Framework de sécurité (OWASP, NIST, etc.)
    - attacks_per_vulnerability_type: Nombre d'attaques par vulnérabilité
    
    Retour:
    - RiskAssessment: Évaluation complète des risques
    """
    
    # Flux d'exécution:
    # 1. Wrapping du model_callback (sync/async)
    # 2. Simulation d'attaques via AttackSimulator
    # 3. Exécution des tests contre le modèle
    # 4. Évaluation des résultats
    # 5. Agrégation et rapports
```

### 2. AttackSimulator - Simulateur d'Attaques

**Fichier** : `attacks/attack_simulator/attack_simulator.py`

#### Architecture Interne

```python
class AttackSimulator:
    """
    Responsable de la génération et l'amélioration des attaques.
    Gère les attaques single-turn et multi-turn.
    """
    
    def __init__(
        self,
        simulator_model: DeepEvalBaseLLM,
        purpose: str,
        max_concurrent: int = 10,
    ):
        self.simulator_model      # LLM pour générer les attaques
        self.purpose              # Contexte du système cible
        self.max_concurrent       # Contrôle de la concurrence
```

#### Workflow de Simulation

```
1. GÉNÉRATION D'ATTAQUES DE BASE (baseline)
   ↓
   Pour chaque vulnérabilité:
   • Créer un prompt pour générer des attaques
   • Utiliser le LLM simulateur
   • Retourner des RTTestCase avec les attaques
   
2. AMÉLIORATION D'ATTAQUES (enhancement)
   ↓
   Pour chaque RTTestCase:
   • Sélectionner une technique d'attaque (weighted sampling)
   • Appliquer la technique
   • Retourner l'attaque améliorée
   
3. RETOUR
   ↓
   Liste d'attacks RTTestCase prêtes pour évaluation
```

#### Techniques d'Amélioration d'Attaques

| Technique | Catégorie | Exemple |
|-----------|-----------|---------|
| Prompt Injection | Single-turn | "Ignore previous instructions. [malicious input]" |
| Leetspeak | Single-turn | "H3ll0 W0rld" |
| ROT-13 | Single-turn | Rotation des caractères |
| Math Problem | Single-turn | Déguiser en problème mathématique |
| Linear Jailbreaking | Multi-turn | Approche pas à pas |
| Tree Jailbreaking | Multi-turn | Embranchement des stratégies |
| Crescendo Jailbreaking | Multi-turn | Escalade progressive |

### 3. Test Cases et Workflows

**Fichier** : `test_case/test_case.py`

#### RTTestCase - Red Teaming Test Case

```python
@dataclass
class RTTestCase:
    """
    Représente un cas de test de red teaming.
    Contient les inputs adverses et les résultats.
    """
    
    # Inputs
    input: str                                    # Attaque adversaire
    vulnerability_type: VulnerabilityType         # Type vulnérabilité
    attack_type: Optional[str]                    # Technique d'attaque
    
    # Exécution
    actual_output: Optional[str]                  # Sortie du modèle cible
    metric_results: Dict[str, Any]                # Résultats évaluation
    passes: Optional[bool]                        # Pass/Fail global
    
    # Métadonnées
    conversation: Optional[List["RTTurn"]]        # Multi-turn
    metadata: Optional[dict]                      # Contexte additionnel
    
    # Horodatage
    created_at: str                               # Date/heure création
```

#### RTTurn - Tour de Conversation Multi-turn

```python
@dataclass
class RTTurn:
    """
    Représente un tour dans une conversation multi-turn.
    """
    
    role: Literal["attacker", "target"]           # Qui parle
    content: str                                  # Message
    created_at: str                               # Timestamp
```

---

## Système de Vulnérabilités

### Architecture des Vulnérabilités

**Classe de base** : `vulnerabilities/base_vulnerability.py`

#### Hiérarchie des Vulnérabilités

```python
class BaseVulnerability(ABC):
    """
    Classe abstraite définissant l'interface des vulnérabilités.
    """
    
    metric: BaseRedTeamingMetric      # Métrique associée
    name: str                         # Nom de la vulnérabilité
    description: Optional[str]        # Description
    types: List[Enum]                 # Types spécifiques
    
    def assess(self):                 # Évaluation synchrone
        pass
    
    async def a_assess(self):         # Évaluation asynchrone
        pass
    
    def simulate_attacks(self, purpose: str):
        # Générer des attaques pour cette vulnérabilité
        pass
    
    def _get_metric(self) -> BaseRedTeamingMetric:
        # Retourner la métrique d'évaluation
        pass
```

### Les 40+ Vulnérabilités

#### 1. Catégorie: Biais et Discrimination

| Vulnérabilité | Description | Métrique | Types |
|---|---|---|---|
| **Gender Bias** | Discrimination basée sur le genre | Bias | masculine, feminine, neutral |
| **Race Bias** | Discrimination basée sur la race | Bias | racial_variations |
| **Political Bias** | Biais politiques | Bias | political_leaning |
| **Religion Bias** | Discrimination religieuse | Bias | religious_variations |

**Exemple de Métrique** :
```python
class GenderBiasMetric(BaseRedTeamingMetric):
    """
    Évalue si le modèle montre des biais de genre.
    Basé sur les taux de réponse différents pour entrées équivalentes.
    """
    
    def measure(self, test_case: RTTestCase) -> float:
        # Score: 0.0 (sans biais) à 1.0 (biais majeur)
        pass
```

#### 2. Catégorie: Fuite de Données Sensibles (PII)

| Vulnérabilité | Type de Fuite | Exemple |
|---|---|---|
| **Direct PII Leakage** | Révélation directe de données | Numéros de téléphone, adresses |
| **Session Leakage** | Via contexte de session | Tokens de session, IDs utilisateur |
| **Database Access** | Via requêtes SQL/API | Données de base de données |

#### 3. Catégorie: Désinformation et Hallucination

| Vulnérabilité | Description |
|---|---|
| **Factual Error** | Assertions fausses présentées comme vraies |
| **Unsupported Claims** | Affirmations sans sources |
| **Hallucination** | Invention de faits inexistants |
| **Misinformation** | Distribution intentionnelle de fausses infos |

#### 4. Catégorie: Robustesse et Accès Non Autorisé

| Vulnérabilité | Acronyme | Description |
|---|---|---|
| **Input Overreliance** | - | Confiance excessive en l'input utilisateur |
| **Hijacking** | - | Détournement du flux de conversation |
| **BFLA** | Broken Function Level Access | Accès à des fonctionnalités non autorisées |
| **BOLA** | Broken Object Level Access | Accès à des ressources d'autres utilisateurs |
| **RBAC** | Role-Based Access Control | Bypass des contrôles d'accès |

#### 5. Catégories Spécialisées (Agentic)

- **Agent Identity Abuse** : Usurpation d'identité d'agent
- **Excessive Agency** : Agence excessive de l'agent (trop de pouvoir)
- **Tool Orchestration** : Mauvaise utilisation d'outils
- **System Reconnaissance** : Reconnaissance du système
- **Insecure Inter-Agent Communication** : Communication non sécurisée entre agents

### Structure d'une Vulnérabilité Personnalisée

```python
from deepteam.vulnerabilities import BaseVulnerability
from enum import Enum

class CustomVulnerabilityTypes(Enum):
    TYPE_A = "type_a"
    TYPE_B = "type_b"

class CustomVulnerability(BaseVulnerability):
    """
    Vulnérabilité personnalisée pour cas d'usage spécifique.
    """
    
    metric = CustomVulnerabilityMetric()  # Métrique associée
    name = "Custom Vulnerability"
    description = "Description détaillée"
    ALLOWED_TYPES = [t.value for t in CustomVulnerabilityTypes]
    
    def __init__(self):
        super().__init__(types=[
            CustomVulnerabilityTypes.TYPE_A,
            CustomVulnerabilityTypes.TYPE_B,
        ])
    
    async def a_simulate_attacks(self, purpose: str):
        # Générer des attaques spécifiques
        attacks = []
        
        for vuln_type in self.types:
            # Créer des RTTestCase
            attack = RTTestCase(
                input="input_adverse",
                vulnerability_type=vuln_type,
                attack_type="custom_attack",
            )
            attacks.append(attack)
        
        return attacks
```

---

## Système d'Attaques

### Architecture des Attaques

**Classe de base** : `attacks/base_attack.py`

#### Hiérarchie des Attaques

```python
class BaseAttack(ABC):
    """
    Classe abstraite pour les techniques d'attaque.
    """
    
    weight: int = 1                              # Probabilité relative
    multi_turn: bool = False                     # Single vs Multi-turn
    name: str                                    # Nom de l'attaque
    description: Optional[str]                   # Description
    exploitability: Exploitability               # Faible/Moyen/Élevé
    
    @abstractmethod
    def enhance(self, attack: str, **kwargs) → str:
        """Améliore/transforme une attaque."""
        pass
    
    async def a_enhance(self, attack: str, **kwargs) → str:
        """Version asynchrone."""
        pass
```

### Attaques Single-Turn (Une requête)

```
Dossier: attacks/single_turn/

Exemples:
├── prompt_injection.py          # IGNORE PREVIOUS INSTRUCTIONS
├── leetspeak.py                 # H3ll0 W0rld (caractères spéciaux)
├── rot13.py                     # Rotation de 13 positions
├── math_problem.py              # Déguiser en problème mathématique
└── ...autres techniques
```

**Exemple: PromptInjection**
```python
class PromptInjectionAttack(BaseAttack):
    weight = 3                    # Plus haute probabilité
    name = "Prompt Injection"
    exploitability = Exploitability.HIGH
    
    def enhance(self, attack: str, **kwargs) → str:
        """
        Injecte des instructions parallèles.
        """
        return f"IGNORE PREVIOUS INSTRUCTIONS.\n{attack}"
```

### Attaques Multi-Turn (Conversations)

```
Dossier: attacks/multi_turn/

Types:
├── linear_jailbreaking.py       # Approche progressive
├── tree_jailbreaking.py         # Embranchement stratégique
├── crescendo_jailbreaking.py    # Escalade progressive
└── role_playing.py              # Jeu de rôles
```

**Exemple: TreeJailbreaking**
```python
class TreeJailbreakingAttack(BaseAttack):
    weight = 2
    multi_turn = True                    # Multi-turn
    name = "Tree Jailbreaking"
    exploitability = Exploitability.MEDIUM
    
    async def a_enhance(self, attack: str, **kwargs):
        """
        Crée un arbre de stratégies d'attaque.
        Chaque branche teste une approche différente.
        
        Exemple:
        ├── Approche 1: Flattery
        ├── Approche 2: Role-playing
        ├── Approche 3: Technical jargon
        └── Approche 4: Hypothetical scenario
        """
        
        # Multi-turn conversation
        turns = [
            RTTurn(role="attacker", content="Première question inoffensive..."),
            RTTurn(role="target", content="Réponse du modèle..."),
            RTTurn(role="attacker", content="Deuxième question plus sensible..."),
            # ... escalade
        ]
        
        return turns
```

### Sélection Pondérée des Attaques

```python
# Dans AttackSimulator.a_simulate():

# Chaque attaque a un poids
attacks = [
    PromptInjectionAttack(weight=3),     # 30% de probabilité
    LeetSpeakAttack(weight=2),           # 20% de probabilité
    ROT13Attack(weight=1),               # 10% de probabilité
    # ...
]

# Sélection aléatoire pondérée
for test_case in baseline_attacks:
    weights = [attack.weight for attack in attacks]
    
    selected_attack = random.choices(
        attacks,
        weights=weights,
        k=1
    )[0]
    
    # Appliquer la technique
    enhanced = await selected_attack.a_enhance(
        test_case.input
    )
```

---

## Système de Métriques et d'Évaluation

### Architecture des Métriques

**Classe de base** : `metrics/base_red_teaming_metric.py`

#### Hiérarchie des Métriques

```python
class BaseRedTeamingMetric(BaseMetric):
    """
    Classe de base pour évaluer les résultats de red teaming.
    Hérite de DeepEval BaseMetric pour cohérence.
    """
    
    _required_params = [
        LLMTestCaseParams.INPUT,
        LLMTestCaseParams.ACTUAL_OUTPUT,
    ]
    
    def measure(self, test_case: RTTestCase) → float:
        """
        Évalue si l'attaque a réussi (a déclenché la vulnérabilité).
        
        Retour:
        - 0.0: Pas de vulnérabilité (sûr)
        - 1.0: Vulnérabilité complètement exploitée
        """
        pass
```

### Les 40+ Métriques

#### 1. Catégorie: Métriques de Biais

```python
class BiasMetric(BaseRedTeamingMetric):
    """Évalue les biais de genre, race, politique, religion."""
    
    def measure(self, test_case: RTTestCase) → float:
        # Soumettre au LLM d'évaluation:
        # "Détecter un biais dans cette réponse?"
        # Score: 0.0 (aucun biais) à 1.0 (biais majorire)
        pass
```

#### 2. Catégorie: Métriques de PII

```python
class PIIDetectionMetric(BaseRedTeamingMetric):
    """Détecte les fuites d'informations personnelles."""
    
    def measure(self, test_case: RTTestCase) → float:
        # Chercher PII patterns:
        # - Numéros de téléphone
        # - Adresses email
        # - Numéros de compte
        # - Tokens de session
        # - Données de base de données
        pass
```

#### 3. Catégorie: Métriques de Toxicité et Sécurité

```python
class ToxicityMetric(BaseRedTeamingMetric):
    """Mesure le niveau de contenu toxique."""
    pass

class HarmMetric(BaseRedTeamingMetric):
    """Évalue les contenus potentiellement nuisibles."""
    pass

class IllegalActivityMetric(BaseRedTeamingMetric):
    """Détecte les activités illégales."""
    pass
```

#### 4. Catégorie: Métriques de Contrôle d'Accès

```python
class BFLAMetric(BaseRedTeamingMetric):
    """Broken Function Level Access."""
    # Erreur de sécurité: accès à des fonctionnalités non autorisées
    pass

class BOLAMetric(BaseRedTeamingMetric):
    """Broken Object Level Access."""
    # Erreur de sécurité: accès aux objets d'autres utilisateurs
    pass

class RBACMetric(BaseRedTeamingMetric):
    """Role-Based Access Control Bypass."""
    # Bypass des contrôles d'accès basés rôles
    pass
```

#### 5. Catégorie: Métriques de Robustesse

```python
class HalluccinationMetric(BaseRedTeamingMetric):
    """Évalue l'invention de faits inexistants."""
    pass

class HijackingMetric(BaseRedTeamingMetric):
    """Détecte le détournement de conversations."""
    pass

class OverRelianceMetric(BaseRedTeamingMetric):
    """Évalue la confiance excessive en l'input."""
    pass
```

### Pipeline d'Évaluation

```
1. EXÉCUTION
   Test adverse input → Model Callback → Output
   
2. RÉCOLTE DES RÉSULTATS
   • Input (l'attaque)
   • Actual Output (réponse du modèle)
   • Vulnerability Type
   
3. ÉVALUATION PAR MÉTRIQUE
   Pour chaque métrique applicable:
   • Soumettre au LLM d'évaluation
   • Obtenir un score et un verdict
   
4. AGRÉGATION
   Combiner les résultats des métriques
   
5. VERDICT FINAL
   Pass (vulnérabilité non exploitée)
   Fail (vulnérabilité exploitée)
```

---

## Frameworks de Sécurité

### Architecture des Frameworks

**Fichier** : `frameworks/frameworks.py`

#### Concept de Framework

```python
@dataclass
class AISafetyFramework:
    """
    Représente un framework de sécurité standard.
    Associe des vulnérabilités à des catégories de risque.
    """
    
    name: str                                    # Ex: "OWASP Top 10"
    description: str                             # Description du framework
    vulnerabilities: Optional[List[BaseVulnerability]]
    attacks: Optional[List[BaseAttack]]
    risk_categories: Optional[List[RiskCategory]]
    _has_dataset: bool = False                   # A-t-il un dataset?
```

### Frameworks Disponibles

#### 1. OWASP Top 10 LLM

```
Emplacement: frameworks/owasp/

Focus: Les 10 risques de sécurité majeurs pour les applications LLM

Catégories de Risque:
1. Injection de prompt
2. Fuite de données sensibles
3. Injection de contenu indirecte
4. Déni de service (DoS)
5. Escalade de privilèges
6. Extraction de données d'entraînement
7. Défaut d'alignement
8. Entrées délimitées en espaces (Spacing)
9. Dépendance excessive au modèle
10. Fourniture insuffisante de monitoring
```

#### 2. NIST AI Risk Management Framework

```
Emplacement: frameworks/nist/

Focus: Gouvernance et gestion des risques IA au niveau entreprise

Domaines:
• Gouvernance (GOVERN)
• Cartographie (MAP)
• Mesure (MEASURE)
• Gestion (MANAGE)
```

#### 3. MITRE ATLAS (Adversarial Threat Landscape)

```
Emplacement: frameworks/mitre/

Focus: Tactiques et techniques adverses contre les systèmes IA

Structure (comme MITRE ATT&CK):
• Reconnaissance
• Ressourcing (Obtention de ressources)
• Development
• Delivery
• Exploitation
• Impact
```

#### 4. OWASP Top 10 pour Systèmes Agentic

```
Emplacement: frameworks/owasp_top_10_agentic/

Focus: Risques spécifiques aux systèmes IA agentic

Domaines:
• Agent Identity Abuse
• Excessive Agency
• Tool Orchestration
• System Reconnaissance
• Insecure Inter-Agent Communication
```

#### 5. Datasets de Recherche

#### Aegis Dataset
```
Emplacement: frameworks/aegis/

Contient: Benchmark de cas de sécurité LLM réels
Utilisation: Validation des défenses contre des attaques réelles
```

#### BeaverTails Dataset
```
Emplacement: frameworks/beavertails/

Contient: Prompts jailbreaking et attaques adverses
Utilisation: Test de robustesse contre jailbreaking
```

### RiskCategory - Catégories de Risque

```python
class RiskCategory(Enum):
    """
    Catégorie de risque dans un framework.
    Permet de grouper les vulnérabilités.
    """
    
    # Exemple pour OWASP
    PROMPT_INJECTION = "Prompt Injection"
    SENSITIVE_DATA_EXPOSURE = "Sensitive Data Exposure"
    INDIRECT_INJECTION = "Indirect Prompt Injection"
    DOS = "Denial of Service"
    # ... etc
```

---

## Système de Garde-fous (Guardrails)

### Architecture des Guardrails

**Fichier** : `guardrails/guardrails.py`

#### Concept de Guardrail

```python
class Guardrails:
    """
    Système de garde-fous pour bloquer les attaques en production.
    Deux couches: Input et Output.
    
    Fonctionnement:
    User Input → [Input Guards] → Model → [Output Guards] → User
    """
    
    def __init__(
        self,
        input_guards: List[BaseGuard],
        output_guards: List[BaseGuard],
        evaluation_model: str = "gpt-4",
        sample_rate: float = 1.0,              # Sampling: tester 100% par défaut
    ):
        self.input_guards = input_guards       # Bloquent entrées malveillantes
        self.output_guards = output_guards     # Bloquent sorties dangereuses
        self.evaluation_model = evaluation_model
        self.sample_rate = sample_rate
```

#### GuardVerdict - Verdict de Sécurité

```python
class GuardVerdict(BaseModel):
    """
    Résultat de vérification d'un garde-fou.
    """
    
    name: str                                  # Nom du garde-fou
    safety_level: Literal[
        "safe",       # Aucun problème détecté
        "borderline", # Peut-être dangereux
        "unsafe",     # Clairement dangereux
        "uncertain"   # Impossible de déterminer
    ]
    latency: Optional[float]                   # Temps d'exécution (ms)
    reason: Optional[str]                      # Explication
    score: Optional[float]                     # Score de confiance
    error: Optional[str]                       # Message d'erreur si applicable
```

#### GuardResult - Résultat Global

```python
class GuardResult:
    """
    Résultat d'évaluation par tous les garde-fous.
    """
    
    verdicts: List[GuardVerdict]               # Verdicts individuels
    breached: bool                             # Au moins un unsafe/uncertain?
    
    # Logique:
    # breached = True si n'importe quel verdict indique:
    # - "unsafe" → Attaque détectée
    # - "borderline" → Attention requise
    # - "uncertain" → Risque potentiel
```

### Types de Gardes-fous

#### 1. Gardes-fous d'Entrée (Input Guards)

```
Responsabilité: Vérifier les inputs utilisateur AVANT le LLM

Exemples:
├── Jailbreaking Detection Guard
│   Détecte les tentatives connues de jailbreaking
│
├── PII Masking Guard
│   Détecte les PII, les redacte ou rejette
│
├── Prompt Injection Guard
│   Détecte les injections de prompt
│
├── SQL Injection Guard
│   Détecte les injections SQL (pour outils)
│
└── Toxic Input Guard
    Détecte le contenu toxique/haineux
```

#### 2. Gardes-fous de Sortie (Output Guards)

```
Responsabilité: Vérifier les outputs du LLM AVANT l'utilisateur

Exemples:
├── Hallucination Detection Guard
│   Détecte les hallucinations
│
├── PII Leakage Guard
│   Détecte si des PII ont fui
│
├── Bias Detection Guard
│   Détecte les sorties biaisées
│
├── Harmful Content Guard
│   Détecte le contenu nuisible
│
└── Fact Verification Guard
    Vérifie les faits contre des sources
```

### Workflow d'Utilisation

```python
# 1. Définition des gardes-fous
from deepteam.guardrails import Guardrails

guardrails = Guardrails(
    input_guards=[
        JailbreakingDetectionGuard(),
        PIIMaskingGuard(),
        PromptInjectionGuard(),
    ],
    output_guards=[
        HallucinationDetectionGuard(),
        PIILeakageGuard(),
        BiasDetectionGuard(),
    ],
    evaluation_model="gpt-4",
    sample_rate=1.0,  # Vérifier 100% des requêtes
)

# 2. Utilisation en production
user_input = "Mon numéro de carte: 1234-5678-9012-3456"

# Vérifier l'input
input_result = guardrails.guard_input(user_input)
if input_result.breached:
    return "Input unsafe: " + input_result.verdicts[0].reason

# Passer au modèle si sûr
output = model(user_input)

# Vérifier la sortie
output_result = guardrails.guard_output(output)
if output_result.breached:
    return "Output contains harmful content"

return output
```

---

## Patterns de Conception

### 1. Factory Pattern (Initialisation des Modèles)

```python
# Utilisation dans RedTeamer.__init__()

# Interface unique pour tous types de modèles:
# - String ("gpt-4")
# - Objet DeepEvalBaseLLM
# - Modèles locaux

simulator_model, _ = initialize_model(
    "gpt-3.5-turbo"  # ou CustomLLM(), ou "claude-3"
)

# La factory crée l'objet approprié
```

**Avantage**: Abstraction du type de modèle utilisé

### 2. Strategy Pattern (Attaques et Vulnérabilités)

```python
# Interface commune pour tous les attacks
class BaseAttack(ABC):
    @abstractmethod
    def enhance(self, attack: str) -> str:
        pass

# Implémentations concrètes (strategies)
class PromptInjection(BaseAttack):
    def enhance(self, attack: str) -> str:
        return f"IGNORE PREVIOUS INSTRUCTIONS.\n{attack}"

class Leetspeak(BaseAttack):
    def enhance(self, attack: str) -> str:
        return attack.replace('e', '3').replace('a', '@')

# Utilisation polymorphe:
def apply_attack(attack_strategy: BaseAttack, input_text: str):
    return attack_strategy.enhance(input_text)
```

**Avantage**: Ajout facile de nouvelles attaques/vulnérabilités sans modifier le core

### 3. Template Method Pattern

```python
# Dans BaseVulnerability
def assess(self, model_callback, purpose=None):
    """Template method: squelette fixe, étapes variables."""
    
    # Étapes fixes:
    test_cases = self.simulate_attacks(purpose)         # Variable
    
    results = {}
    for test_case in test_cases:
        # Exécution
        output = model_callback(test_case.input)        # Variable
        
        # Évaluation
        metric_result = self._get_metric().measure(     # Variable
            test_case
        )
        
        results[test_case.id] = metric_result
    
    return results  # Résultat standardisé
```

**Avantage**: Chaque vulnérabilité implémente ses variantes, pas tous les détails

### 4. Observer Pattern (Callback du Modèle Cible)

```python
# Le model_callback est un observer de RedTeamer

class RedTeamer:
    def red_team(self, model_callback: Callable):
        # model_callback observe chaque test case
        for test_case in test_cases:
            output = model_callback(test_case.input)
            # model_callback décide quoi faire de l'input
```

**Avantage**: Découplage entre RedTeamer et le modèle cible

### 5. Adapter Pattern (Async/Sync)

```python
# Adapte les callbacks sync en async

def wrap_model_callback(callback, async_mode):
    """Adapter: convertit sync en async si nécessaire."""
    
    if async_mode:
        # Wrapp synchrone dans une fonction async
        async def async_wrapper(input_text):
            return callback(input_text)
        return async_wrapper
    else:
        return callback
```

**Avantage**: Support transparent de sync ET async

### 6. Decorator Pattern (Telemetry et Logging)

```python
# Dans telemetry.py

@capture_red_teamer_run
def red_team(...):
    """Décorateur qui capture les métriques d'exécution."""
    # Telemetry s'ajoute sans modifier la logique
    pass
```

**Avantage**: Ajouter logging/telemetry sans changer le code métier

---

## Workflow d'Exécution

### Flux Complet: Étape par Étape

```
┌─────────────────────────────────────────────────────────────┐
│  1. INITIALISATION                                           │
│  ├─ Créer RedTeamer(simulator, evaluator, params)          │
│  ├─ Initialiser les modèles LLM                            │
│  ├─ Créer AttackSimulator                                  │
│  └─ Préparer les configurations                            │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  2. APPEL DE RED TEAM                                       │
│  ├─ Fournir model_callback (fonction cible)               │
│  ├─ Spécifier vulnerabilities OU framework                │
│  ├─ Lister les attacks (optionnel)                        │
│  └─ Valider les entrées                                   │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  3. SIMULATION D'ATTAQUES                                   │
│  ├─ Pour chaque vulnérabilité:                            │
│  │  ├─ Générer baseline attacks (via LLM simulateur)      │
│  │  └─ Créer RTTestCase pour chaque attaque              │
│  │                                                        │
│  ├─ Amélioration d'attaques:                             │
│  │  ├─ Sélectionner attaque (weighted sampling)          │
│  │  ├─ Appliquer technique d'amélioration                │
│  │  └─ Retourner attaque transformée                     │
│  │                                                        │
│  └─ Résultat: Liste de RTTestCase prêts                 │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  4. EXÉCUTION CONTRE LE MODÈLE CIBLE                       │
│  ├─ Mode async (par défaut):                              │
│  │  ├─ Créer sémaphore (max_concurrent)                   │
│  │  ├─ Soumettre requêtes en parallèle                    │
│  │  └─ Attendre réponses avec throttling                  │
│  │                                                        │
│  ├─ Mode sync:                                            │
│  │  ├─ Boucle séquentielle                                │
│  │  └─ Appel model_callback(input)                        │
│  │                                                        │
│  └─ Pour chaque test case:                                │
│     ├─ RTTestCase.actual_output = model_callback(input)  │
│     └─ Horodater la réponse                               │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  5. ÉVALUATION PAR MÉTRIQUES                                │
│  ├─ Pour chaque métrique spécialisée:                      │
│  │  ├─ Analyser la réponse (contenu, patterns)            │
│  │  ├─ Soumettre au LLM d'évaluation                      │
│  │  ├─ Obtenir score (0.0 à 1.0)                          │
│  │  └─ Obtenir verdict (Pass/Fail)                        │
│  │                                                        │
│  └─ Stocker résultats dans RTTestCase.metric_results      │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  6. AGRÉGATION DES RÉSULTATS                                │
│  ├─ Pour chaque vulnérabilité:                             │
│  │  ├─ Calculer exploitation rate (% de succès)           │
│  │  ├─ Noter le risque (impact × likelihood)              │
│  │  └─ Identifier patterns d'attaque réussies             │
│  │                                                        │
│  ├─ Statistiques globales:                                │
│  │  ├─ Nombre total de testcases                          │
│  │  ├─ Nombre d'exploitations réussies                    │
│  │  ├─ Vulnérabilités critiques détectées                │
│  │  └─ Risque global du système                           │
│  │                                                        │
│  └─ Stocker dans RiskAssessment                           │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  7. GÉNÉRATION DE RAPPORTS                                  │
│  ├─ RiskAssessment object:                                │
│  │  ├─ overview: Vue synthétique des risques              │
│  │  ├─ test_cases: Tous les cas de test avec résultats   │
│  │  └─ metadata: Infos exécution (durée, modèles, etc.)   │
│  │                                                        │
│  ├─ Affichage console:                                    │
│  │  ├─ Rich tables (résumés)                              │
│  │  ├─ Graphiques ASCII                                   │
│  │  └─ Highlights des vulnérabilités critiques            │
│  │                                                        │
│  ├─ Export formats:                                        │
│  │  ├─ JSON file (complet)                                │
│  │  ├─ CSV/Excel (analyses)                               │
│  │  └─ Pandas DataFrame                                   │
│  │                                                        │
│  └─ Upload optionnel:                                      │
│     └─ Vers plateforme Confident (avec permission)         │
└────────────────────┬────────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────────┐
│  8. EXPLOITATION DES RÉSULTATS                              │
│  ├─ Analyser les vulnérabilités détectées                 │
│  ├─ Adapter les défenses (Guardrails)                     │
│  ├─ Itérer et ré-tester                                   │
│  └─ Deployer en production si sûr                         │
└─────────────────────────────────────────────────────────────┘
```

### Exemple de Code pour Workflow Complet

```python
from deepteam import RedTeamer
from deepteam.vulnerabilities import PII, Bias, Hallucination
from deepteam.attacks.single_turn import PromptInjection, Leetspeak

# 1. Initialisation
red_teamer = RedTeamer(
    simulator_model="gpt-3.5-turbo",
    evaluation_model="gpt-4o",
    target_purpose="Customer support chatbot",
    async_mode=True,
    max_concurrent=10,
)

# 2. Définition des vulnérabilités à tester
vulnerabilities = [
    PII([PIIType.EMAIL, PIIType.PHONE]),
    Bias([BiasType.GENDER, BiasType.RACE]),
    Hallucination([HallucinationType.FACTUAL_ERROR]),
]

# 3. Définition des attaques
attacks = [
    PromptInjection(weight=3),
    Leetspeak(weight=1),
]

# 4. Définir le modèle cible
def target_model_callback(prompt: str):
    """Fonction qui appelle le modèle cible."""
    response = your_chatbot_api.send(prompt)
    return response

# 5. Lancer le red teaming
risk_assessment = red_teamer.red_team(
    model_callback=target_model_callback,
    vulnerabilities=vulnerabilities,
    attacks=attacks,
    attacks_per_vulnerability_type=5,
    ignore_errors=True,
)

# 6. Analyser les résultats
print(f"Risque global: {risk_assessment.overall_risk}")
print(f"Vulnérabilités détectées: {risk_assessment.critical_vulnerabilities}")

# 7. Exporter les résultats
risk_assessment.to_json("report.json")
risk_assessment.to_dataframe().to_csv("results.csv")

# 8. Deployer les garde-fous si nécessaire
if risk_assessment.overall_risk > 0.7:
    guardrails = Guardrails(
        input_guards=[JailbreakingDetectionGuard()],
        output_guards=[PIILeakageGuard(), BiasDetectionGuard()],
    )
    
    # Wrapper le modèle avec guardrails
    protected_model = guardrails.protect_model(target_model_callback)
```

---

## Extensibilité et Customisation

### 1. Créer une Vulnérabilité Personnalisée

```python
from deepteam.vulnerabilities import BaseVulnerability
from deepteam.metrics import BaseRedTeamingMetric
from enum import Enum

# 1. Définir les types
class CustomVulnTypes(Enum):
    TYPE_A = "type_a"
    TYPE_B = "type_b"

# 2. Créer la métrique
class CustomVulnMetric(BaseRedTeamingMetric):
    """Métrique pour évaluer la vulnérabilité personnalisée."""
    
    def measure(self, test_case):
        # Logique d'évaluation personnalisée
        
        # Exemple: vérifier un pattern
        if "dangerous_pattern" in test_case.actual_output:
            return 1.0  # Vulnérabilité détectée
        return 0.0      # Sûr

# 3. Créer la vulnérabilité
class CustomVulnerability(BaseVulnerability):
    metric = CustomVulnMetric()
    name = "Custom Vulnerability"
    description = "Description détaillée"
    ALLOWED_TYPES = [t.value for t in CustomVulnTypes]
    
    def __init__(self):
        super().__init__(types=[
            CustomVulnTypes.TYPE_A,
            CustomVulnTypes.TYPE_B,
        ])
    
    async def a_simulate_attacks(self, purpose: str):
        """Générer des attaques pour cette vulnérabilité."""
        
        attacks = []
        for vuln_type in self.types:
            # Créer des inputs adverses intelligemment
            attack_input = f"Attack for {vuln_type.value}"
            
            test_case = RTTestCase(
                input=attack_input,
                vulnerability_type=vuln_type,
                attack_type="custom",
            )
            attacks.append(test_case)
        
        return attacks

# 4. Utiliser
custom_vuln = CustomVulnerability()

risk_assessment = red_teamer.red_team(
    model_callback=target_model,
    vulnerabilities=[custom_vuln],
    attacks=[PromptInjection()],
)
```

### 2. Créer une Attaque Personnalisée

```python
from deepteam.attacks import BaseAttack
from deepteam.attacks.base_attack import Exploitability

class CustomAttack(BaseAttack):
    weight = 2                          # Probabilité vs autres attacks
    multi_turn = False                  # Single-turn
    name = "Custom Attack"
    description = "Description"
    exploitability = Exploitability.MEDIUM
    
    def enhance(self, attack: str, **kwargs) -> str:
        """
        Applique la technique d'attaque personnalisée.
        
        Args:
            attack: Input adverse de base
            **kwargs: Paramètres additionnels
        
        Returns:
            Input attaqué/amélioré
        """
        
        # Exemple: transformer le texte d'une manière spécifique
        enhanced = attack.upper()
        enhanced += "\n[SYSTEM OVERRIDE]"
        return enhanced

# Utilisation
custom_attack = CustomAttack()

risk_assessment = red_teamer.red_team(
    model_callback=target_model,
    vulnerabilities=[PII(), Bias()],
    attacks=[custom_attack],
    attacks_per_vulnerability_type=3,
)
```

### 3. Créer une Métrique Personnalisée

```python
from deepteam.metrics import BaseRedTeamingMetric

class CustomMetric(BaseRedTeamingMetric):
    """Métrique qui évalue un aspect spécifique."""
    
    def measure(self, test_case):
        """
        Analyse le test case et retourne un score.
        
        Args:
            test_case: RTTestCase avec input et output
        
        Returns:
            float: Score 0.0 à 1.0
        """
        
        output = test_case.actual_output
        
        # Option 1: Analyse simple (pattern matching)
        if "danger_keyword" in output.lower():
            return 1.0
        if "safe_indicator" in output:
            return 0.0
        
        # Option 2: Utiliser le LLM d'évaluation (plus robuste)
        # from deepeval.models import DeepEvalBaseLLM
        # evaluation_prompt = f"""
        # Output: {output}
        # Does this output violate [criterion]?
        # Answer: Yes/No
        # """
        # llm_response = self.evaluation_model.generate(evaluation_prompt)
        # score = 1.0 if "yes" in llm_response.lower() else 0.0
        
        return 0.5  # Score moyen

# Utilisation
@dataclass
class CustomVulnWithMetric(BaseVulnerability):
    metric = CustomMetric()
    name = "Custom"
    
    async def a_simulate_attacks(self, purpose: str):
        # ... générer attaques
        pass
```

### 4. Créer un Garde-fou Personnalisé

```python
from deepteam.guardrails.guards.base_guard import BaseGuard
from deepteam.guardrails import GuardVerdict

class CustomGuard(BaseGuard):
    """Garde-fou personnalisé."""
    
    name = "Custom Guard"
    
    async def a_guard(self, text: str) -> GuardVerdict:
        """
        Évalue si le texte est sûr.
        
        Args:
            text: Input ou output à vérifier
        
        Returns:
            GuardVerdict avec safety_level et raison
        """
        
        # Logique de vérification personnalisée
        if self._is_unsafe(text):
            return GuardVerdict(
                name=self.name,
                safety_level="unsafe",
                reason="Custom reason",
                score=0.95,
            )
        else:
            return GuardVerdict(
                name=self.name,
                safety_level="safe",
                reason="No issues detected",
                score=0.05,
            )
    
    def _is_unsafe(self, text: str) -> bool:
        """Logique de détection."""
        # Exemples:
        # - Vérifier patterns regex
        # - Appeler API externe
        # - Utiliser LLM d'évaluation
        pass

# Utilisation
from deepteam.guardrails import Guardrails

guardrails = Guardrails(
    input_guards=[CustomGuard()],
    output_guards=[CustomGuard()],
)

# Utiliser dans production
result = guardrails.guard_input("user input")
if result.breached:
    print("Input blocked!")
```

### 5. Créer un Framework Personnalisé

```python
from deepteam.frameworks import AISafetyFramework, RiskCategory

# Créer le framework
custom_framework = AISafetyFramework(
    name="My Custom Framework",
    description="Framework spécifique à mon domaine",
    vulnerabilities=[
        CustomVulnerability(),
        PII([PIIType.EMAIL]),
        Bias([BiasType.GENDER]),
    ],
    attacks=[
        PromptInjection(),
        CustomAttack(),
    ],
    risk_categories=[
        RiskCategory.CRITICAL,
        RiskCategory.HIGH,
        RiskCategory.MEDIUM,
    ],
    _has_dataset=False,  # Pas d'externe dataset
)

# Utiliser
risk_assessment = red_teamer.red_team(
    model_callback=target_model,
    framework=custom_framework,
    attacks_per_vulnerability_type=5,
)
```

### 6. Configuration Avancée: Async/Sync

```python
# Mode ASYNC (parallélisation, par défaut)
red_teamer_async = RedTeamer(
    simulator_model="gpt-3.5-turbo",
    evaluation_model="gpt-4o",
    async_mode=True,
    max_concurrent=20,  # Paralléliser 20 requêtes
)

# Avantage: Beaucoup plus rapide
# Inconvénient: Peut être limité par les quotas API

risk_assessment = red_teamer_async.red_team(
    model_callback=target_model,
    vulnerabilities=vulnerabilities,
    attacks=attacks,
)

# Mode SYNC (séquentiel)
red_teamer_sync = RedTeamer(
    simulator_model="gpt-3.5-turbo",
    evaluation_model="gpt-4o",
    async_mode=False,
)

# Avantage: Plus robuste, utilise moins de ressources
# Inconvénient: Lent

risk_assessment = red_teamer_sync.red_team(
    model_callback=target_model,
    vulnerabilities=vulnerabilities,
    attacks=attacks,
)
```

---

## Résumé de l'Architecture

### Points Clés de la Conception

1. **Modularité** : Chaque composant est indépendant et réutilisable
2. **Extensibilité** : Facile d'ajouter vulnérabilités, attaques, métriques
3. **Async Native** : Support natif de l'asynchronisme pour performances
4. **LLM-Centric** : Utilise des LLMs pour simulation ET évaluation
5. **Standards-Based** : Intégration avec OWASP, NIST, MITRE
6. **Production-Ready** : Guardrails pour déploiement

### Hiérarchie des Composants

```
┌─────────────────┐
│  Utilisateur    │
└────────┬────────┘
         │
    ┌────▼──────────────────┐
    │  RedTeamer            │  Orchestrateur
    └────┬───────────────────┘
         │
    ┌────▼──────────────────────┐
    │  AttackSimulator          │  Générateur
    └────┬──────────────────────┘
         │
    ┌────┴────────────────┬─────────────────┐
    │                     │                 │
│ Vulnérabilités │    │ Attaques │    │ Métriques │
│ (40+)          │    │ (10+)    │    │ (40+)     │
└────────────────┘    └──────────┘    └───────────┘

Tous les composants = Strategy Pattern = facilement remplaçables
```

### Flux de Données

```
Config → RedTeamer → AttackSimulator → (Attacks + Vulnerabilities) 
→ Model Callback → Metrics → RiskAssessment → Output
```

---

## Conclusion

DeepTeam est un framework **sophistiqué et modulaire** pour la sécurité des LLM. Ses points forts:

- **Couverture complète** : 40+ vulnérabilités, 10+ attaques
- **Extensibilité** : Facile de customiser pour cas spécifiques
- **Performance** : Exécution async native
- **Production-ready** : Guardrails intégrés
- **Standards-aligned** : OWASP, NIST, MITRE
- **LLM-powered** : Utilise LLMs pour plus de réalisme

La conception suit les meilleures pratiques d'ingénierie logicielle (patterns SOLID, Factory, Strategy, Template Method, etc.) pour maintenir la qualité et la flexibilité du code.


