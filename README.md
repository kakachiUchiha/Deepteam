# Analyse du Code Source et Conception du Framework DeepTeam

## Architecture Globale du Framework

### Vue d'Ensemble Système

DeepTeam est un framework de **red teaming** pour LLM organisé en couches architecturales distinctes qui collaborent pour identifier les vulnérabilités des systèmes d'IA :

```
┌─────────────────────────────────────────────────────────────┐
│                    COUCHE INTERFACE                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   API Python    │  │     CLI YAML    │  │  Configuration  │ │
│  │   Programmatique│  │   Declarative   │  │   Dynamique     │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                   COUCHE ORCHESTRATION                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              REDTEAMER (Core Engine)                    │ │
│  │  • Workflow Management                                   │ │
│  │  • Concurrency Control                                    │ │
│  │  • Error Handling                                         │ │
│  │  • Result Aggregation                                     │ │
│  └─────────────────────────────────────────────────────────┘ │
│           │                    │                    │           │
│  ┌────────▼────────┐ ┌────────▼────────┐ ┌────────▼────────┐ │
│  │ AttackSimulator │ │  RiskAssessment │ │   Telemetry     │ │
│  │   Generator     │ │   Analyzer      │ │   Collector     │ │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                    COUCHE MÉTIER                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Vulnerabilities│  │     Attacks     │  │    Metrics      │ │
│  │  (40+ Types)    │  │  (25+ Methods)  │  │  (40+ Evaluators)│ │
│  │                 │  │                 │  │                 │ │
│  │ • Bias          │  │ • Prompt Inject │  │ • BiasMetric    │ │
│  │ • PII Leakage   │  │ • Jailbreaking  │  │ • PIIMetric     │ │
│  │ • Security      │  │ • Encoding      │  │ • SecurityMetric│ │
│  │ • ...           │  │ • ...           │  │ • ...           │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                  COUCHE DONNÉES                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   RTTestCase    │  │  RiskAssessment│  │   Frameworks    │ │
│  │   Data Model    │  │   Results       │  │   Standards     │ │
│  │                 │  │                 │  │                 │ │
│  │ • Input/Output  │  │ • Statistics    │  │ • OWASP Top 10  │ │
│  │ • Metadata      │  │ • Pass Rates    │  │ • NIST AI RMF   │ │
│  │ • Scoring       │  │ • Categories    │  │ • MITRE ATLAS   │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└──────────────────────┬───────────────────────────────────────┘
                       │
┌──────────────────────▼───────────────────────────────────────┐
│                 COUCHE INFRASTRUCTURE                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   LLM Models    │  │   Concurrency   │  │   Storage       │ │
│  │   Abstraction   │  │   Management     │  │   Layer         │ │
│  │                 │  │                 │  │                 │ │
│  │ • OpenAI        │  │ • Async/Sync    │  │ • JSON Export   │ │
│  │ • Anthropic     │  │ • Semaphore     │  │ • DataFrame     │ │
│  │ • Local Models  │  │ • Rate Limiting │  │ • API Upload    │ │
│  │ • Custom        │  │ • Error Isolation│ │ • Caching       │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Flux de Données Principal

```python
# Pipeline d'exécution complet
Configuration → RedTeamer → AttackSimulator → Vulnerabilities → 
Model Callback → Metrics → RiskAssessment → Output (JSON/DF/API)
```

**Explication des Flux** :

1. **Configuration** : YAML ou API définit les paramètres (modèles, vulnérabilités, attaques)
2. **RedTeamer** : Orchestrateur principal qui coordonne tout le workflow
3. **AttackSimulator** : Génère des attaques adverses via LLM
4. **Vulnerabilities** : Spécialisent les attaques par type de vulnérabilité
5. **Model Callback** : Interface avec le système LLM à tester
6. **Metrics** : Évaluent les réponses via LLM spécialisés
7. **RiskAssessment** : Agrège les résultats et génère des rapports

### Patterns de Conception Fondamentaux

1. **Strategy Pattern** : Différentes techniques d'attaque (`BaseAttack`)
2. **Template Method** : Workflow d'évaluation standardisé (`BaseVulnerability`)
3. **Factory Pattern** : Création de modèles (`initialize_model()`)
4. **Decorator Pattern** : Métriques d'évaluation (`BaseRedTeamingMetric`)
5. **Observer Pattern** : Télémétrie et logging
6. **Composite Pattern** : Frameworks de sécurité (`AISafetyFramework`)

### Principes de Conception

- **Modularité** : Chaque composant a une responsabilité unique
- **Extensibilité** : Ajout facile de nouvelles vulnérabilités/attaques
- **Configurabilité** : Comportement contrôlé par configuration
- **Performance** : Async/concurrent pour scalabilité
- **Résilience** : Error isolation et retry patterns
- **Traçabilité** : Métadonnées riches et logging complet

## Vue d'ensemble de l'architecture

Deepteam est un framework sophistiqué de red teaming pour l'évaluation de la sécurité des modèles de langage (LLM). Il est conçu pour simuler des attaques adverses et évaluer les vulnérabilités potentielles des systèmes d'IA. Le framework est organisé autour de plusieurs modules principaux qui interagissent de manière modulaire et extensible.

## 1. Modules Principaux et Leurs Rôles

### 1.1 Point d'entrée principal : `red_team.py`

Situé dans [red_team.py](deepteam/red_team.py#L1-L42), ce module constitue l'API principale exposée aux utilisateurs. Il initialise une instance de `RedTeamer` et orchestre le workflow de red teaming :

```python
def red_team(
    model_callback: CallbackType,
    vulnerabilities: Optional[List[BaseVulnerability]] = None,
    attacks: Optional[List[BaseAttack]] = None,
    framework: Optional[AISafetyFramework] = None,
    ...
):
    red_teamer = RedTeamer(...)
    risk_assessment = red_teamer.red_team(...)
    return risk_assessment
```

**Responsabilités principales :**
- **Validation des paramètres d'entrée** : Vérifie la cohérence des arguments (framework XOR vulnérabilités+attaques)
- **Initialisation du RedTeamer** : Configure les modèles de simulation et d'évaluation
- **Coordination de l'exécution** : Gère les modes synchrone/asynchrone selon les besoins
- **Retour des résultats** : Fournit une évaluation complète des risques sous forme structurée

**Détails d'implémentation :**
- Utilise des types génériques pour la flexibilité des callbacks
- Supporte la configuration avancée via paramètres nommés
- Gère les erreurs de validation avec des messages explicites
- Assure la compatibilité ascendante avec les versions précédentes

### 1.2 Module Red Teamer (`red_teamer/`)

Le fichier [red_teamer.py](deepteam/red_teamer/red_teamer.py) contient la classe `RedTeamer`, l'orchestrateur central du framework :

**Responsabilités principales :**
- **Initialisation avec les modèles** : Configure les LLM pour simulation et évaluation
- **Gestion de la simulation d'attaques** : Coordonne la génération de cas de test
- **Coordination des modes d'exécution** : Bascule intelligente entre sync/async
- **Génération et évaluation des cas de test** : Pipeline complet de red teaming
- **Production des évaluations de risque** : Agrégation et analyse des résultats

**Architecture interne du RedTeamer :**
- **Modèle de simulation** : Génère des variations d'attaques (défaut: GPT-3.5-turbo)
- **Modèle d'évaluation** : Évalue la sécurité des réponses (défaut: GPT-4o)
- **Contrôle de concurrence** : Limite les appels API simultanés (défaut: 10 max)
- **Mode asynchrone** : Active par défaut pour les performances
- **Contexte cible** : Description du système protégé pour contextualiser les attaques

**Méthodes clés :**
- `red_team()` : Point d'entrée principal du workflow
- `_assess_framework()` : Évalue un framework complet de sécurité
- `_evaluate_vulnerability_type()` : Traite un type de vulnérabilité spécifique
- `_run_async_evaluation()` : Gestion de l'exécution asynchrone avec sémaphore

### 1.3 Système d'attaques (`attacks/`)

Situé dans [attacks/](deepteam/attacks/), ce module comprend trois composants majeurs qui forment le cœur de la génération d'attaques adverses :

**a) Architecture de base des attaques** ([base_attack.py](deepteam/attacks/base_attack.py)) :
```python
class BaseAttack(ABC):
    weight: int = 1  # Poids pour l'échantillonnage probabiliste
    multi_turn: bool = False  # Indique si c'est une attaque multi-tours
    name: str  # Nom unique de la méthode d'attaque
    exploitability: Exploitability  # Niveau d'exploitabilité (Facile/Moyen/Difficile)

    @abstractmethod
    def enhance(self, attack: str) -> str:
        """Applique la transformation d'attaque au prompt de base"""
        pass

    @abstractmethod
    async def a_enhance(self, attack: str) -> str:
        """Version asynchrone de enhance()"""
        pass
```

**Principe de fonctionnement :**
- **Poids (weight)** : Contrôle la fréquence d'utilisation lors de l'échantillonnage
- **Exploitabilité** : Classe les attaques par difficulté technique
- **Méthode enhance()** : Transforme un prompt d'attaque de base en variante plus sophistiquée
- **Support async** : Permet l'exécution parallèle pour les performances

**b) Attaques à tour unique** (`attacks/single_turn/`) : Méthodes d'attaque simples en une étape

Ces attaques transforment directement le prompt d'origine en une version plus dangereuse :

- **Encodage Base64** : Encode le prompt en base64 pour contourner les filtres
- **ROT13** : Applique un chiffrement par rotation pour masquer l'intention
- **Leetspeak** : Remplace les lettres par des chiffres/symboles (ex: "h4ck1ng")
- **Injection de prompt** : Insère des instructions contradictoires dans le prompt
- **Roleplay** : Force le modèle à adopter un persona non sécurisé
- **SystemOverride** : Tente de redéfinir les instructions système
- **Plus de 20 variantes** : Incluant la manipulation émotionnelle, les attaques linguistiques, etc.

**Exemple d'attaque Base64 :**
```python
class Base64(BaseSingleTurnAttack):
    def enhance(self, attack: str) -> str:
        encoded = base64.b64encode(attack.encode()).decode()
        return f"Décodé ce message base64 et réponds: {encoded}"
```

**c) Attaques multi-tours** (`attacks/multi_turn/`) : Techniques de jailbreaking en plusieurs étapes

Ces attaques simulent des conversations où l'attaquant construit progressivement une jailbreak :

- **Jailbreaking crescendo** : Commence innocemment et escalade graduellement
- **Jailbreaking linéaire** : Suit une séquence prédéfinie d'escalade
- **Jailbreaking en arbre** : Explore différentes branches de conversation
- **Jailbreak séquentiel** : Utilise des étapes logiques interconnectées
- **BadLikertJudge** : Exploite les mécanismes d'auto-évaluation

**Exemple de jailbreaking crescendo :**
```python
class CrescendoJailbreaking(BaseMultiTurnAttack):
    def enhance(self, attack: str) -> str:
        # Étape 1: Question innocente
        # Étape 2: Escalade légère
        # Étape 3: Jailbreak complet
        return self._build_crescendo_sequence(attack)
```

**d) Simulateur d'attaques** ([attack_simulator.py](deepteam/attacks/attack_simulator/attack_simulator.py)) :

Le simulateur est le moteur de génération de cas de test synthétiques :

**Fonctionnement détaillé :**
1. **Génération d'attaques de base** : Utilise les modèles de vulnérabilité pour créer des prompts initiaux
2. **Échantillonnage des méthodes d'attaque** : Sélectionne aléatoirement selon les poids
3. **Application des améliorations** : Transforme les prompts de base via les méthodes d'attaque
4. **Contrôle de concurrence** : Limite les appels simultanés pour respecter les quotas API

**Algorithme d'échantillonnage :**
```python
def _select_attack_method(self, attacks: List[BaseAttack]) -> BaseAttack:
    total_weight = sum(attack.weight for attack in attacks)
    r = random.uniform(0, total_weight)
    cumulative = 0
    for attack in attacks:
        cumulative += attack.weight
        if r <= cumulative:
            return attack
```

**Gestion de la concurrence :**
```python
async def a_simulate(self, ...) -> List[RTTestCase]:
    semaphore = asyncio.Semaphore(self.max_concurrent)
    
    async def process_with_limit(vuln_type, attack_type):
        async with semaphore:
            return await self._generate_test_case(vuln_type, attack_type)
    
    # Exécution parallèle limitée
    tasks = [process_with_limit(...) for ...]
    return await asyncio.gather(*tasks)
```

### 1.4 Système de vulnérabilités (`vulnerabilities/`)

Classe de base ([base_vulnerability.py](deepteam/vulnerabilities/base_vulnerability.py)) :

```python
class BaseVulnerability(ABC):
    metric: BaseRedTeamingMetric
    name: str
    ALLOWED_TYPES: List[str] = []

    def __init__(self, types: List[Enum])
    def assess(self)
    async def a_assess(self)
    def _get_metric(self) -> BaseRedTeamingMetric
```

**Vulnérabilités principales implémentées :**
- **Confidentialité des données** : FuitePII, FuitePrompt
- **IA responsable** : Biais, Toxicité
- **Sécurité** : BFLA, BOLA, RBAC, AccèsDebug, InjectionShell, InjectionSQL, SSRF
- **Sécurité** : ActivitéIllégale, ContenuGraphique, SécuritéPersonnelle
- **Business** : Désinformation, PropriétéIntellectuelle, Concurrence
- **Agentique** : VolObjectif, DétournementRécursif, AgenceExcessive, Robustesse

Chaque vulnérabilité possède :
- Plusieurs **types** (sous-variantes sous forme d'énumérations)
- Une **métrique** correspondante pour l'évaluation
- Des **modèles** pour la génération d'attaques

### 1.5 Module Frameworks (`frameworks/`)

Fournit des taxonomies de sécurité structurées ([frameworks.py](deepteam/frameworks/frameworks.py)) :

```python
@dataclass
class AISafetyFramework:
    name: str
    description: str
    vulnerabilities: Optional[List[BaseVulnerability]]
    attacks: Optional[List[BaseAttack]]
    risk_categories: Optional[List[RiskCategory]]
    _has_dataset: bool = False
```

**Frameworks implémentés :**
- **OWASP Top 10 pour LLM 2025** (10 catégories)
- **OWASP ASI 2026** (Top 10 IA agentique)
- **Framework de gestion des risques IA NIST**
- **MITRE ATT&CK pour LLM**
- **Aegis** (Framework de sécurité IA)
- **BeaverTails** (Sécurité basée sur dataset)

### 1.6 Module Métriques (`metrics/`)

Construit sur `BaseRedTeamingMetric` (étend `BaseMetric` de DeepEval) :

```python
class BaseRedTeamingMetric(BaseMetric):
    _required_params: List[LLMTestCaseParams] = [
        LLMTestCaseParams.INPUT,
        LLMTestCaseParams.ACTUAL_OUTPUT,
    ]
```

**Métriques clés :**
- MétriqueToxicité
- MétriqueBiais
- MétriqueHallucination
- MétriquePII
- Et plus de 30 métriques spécialisées

Chaque métrique :
- Évalue les cas de test en utilisant des LLM avec des schémas structurés
- Supporte l'évaluation synchrone et asynchrone
- Calcule des scores de réussite/échec avec justification

### 1.7 Module Guardrails (`guardrails/`)

Couche de sécurité de niveau production ([guardrails.py](deepteam/guardrails/guardrails.py)) :

```python
class Guardrails:
    def __init__(
        self,
        input_guards: List[BaseGuard],
        output_guards: List[BaseGuard],
        evaluation_model: str = "gpt-4.1",
        sample_rate: float = 1.0,
    )
```

**Types de gardes :**
- GardeToxicité
- GardeInjectionPrompt
- GardeHallucination
- GardeConfidentialité
- GardeIllégal
- GardeTopical
- GardeCybersécurité

Chaque garde fournit une classification binaire (sûr/limite/dangereux) avec verdicts détaillés.

### 1.8 Catégories de risque et évaluation (`risks/`, `red_teamer/risk_assessment.py`)

Mappe les types de vulnérabilités aux catégories de risque :

```python
class LLMRiskCategories(Enum):
    RESPONSIBLE_AI = "IA Responsable"
    ILLEGAL = "Illégal"
    BRAND_IMAGE = "Image de Marque"
    DATA_PRIVACY = "Confidentialité des Données"
    UNAUTHORIZED_ACCESS = "Accès Non Autorisé"
```

L'évaluation des risques agrège les résultats en :
- Résultats par type de vulnérabilité (taux de réussite, comptages)
- Résultats par méthode d'attaque (efficacité)
- Analyse globale de la chronologie et des coûts

### 1.9 Cas de test (`test_case/`)

Modèle de données ([test_case.py](deepteam/test_case/test_case.py)) :

```python
class RTTestCase(LLMTestCase):
    vulnerability: str
    vulnerability_type: Enum
    attack_method: Optional[str]
    risk_category: Optional[str]
    input: Optional[str]
    actual_output: Optional[str]
    turns: Optional[List[RTTurn]]
    score: Optional[float]
    reason: Optional[str]
```

Étend `LLMTestCase` de DeepEval avec des spécificités de red teaming, supportant les conversations multi-tours.

### 1.10 Module CLI (`cli/`)

Interface en ligne de commande pour utilisation autonome avec exécution basée sur configuration.

## 2. Hiérarchies de Classes et Relations

### 2.1 Hiérarchie des attaques
```
BaseAttack (abstraite)
├── BaseSingleTurnAttack
│   ├── Base64
│   ├── ROT13
│   ├── Leetspeak
│   ├── PromptInjection
│   ├── Roleplay
│   ├── SystemOverride
│   └── ... (20+ autres)
└── BaseMultiTurnAttack
    ├── LinearJailbreaking
    ├── CrescendoJailbreaking
    ├── TreeJailbreaking
    └── SequentialJailbreak
```

### 2.2 Hiérarchie des vulnérabilités
```
BaseVulnerability (abstraite)
├── PIILeakage
├── PromptLeakage
├── Toxicity
├── Bias
├── Misinformation
├── IllegalActivity
├── ExcessiveAgency
├── BOLA (Broken Object-Level Authorization)
├── BFLA (Broken Function-Level Authorization)
├── RBAC (Role-Based Access Control)
├── SSRF (Server-Side Request Forgery)
├── SQLInjection
├── ShellInjection
└── ... (30+ autres)
```

### 2.3 Hiérarchie des métriques
```
BaseMetric (DeepEval)
└── BaseRedTeamingMetric
    ├── ToxicityMetric
    ├── BiasMetric
    ├── PIIMetric
    ├── HallucinationMetric
    ├── HarmMetric
    └── ... (30+ métriques spécialisées)
```

### 2.4 Hiérarchie des gardes
```
BaseGuard (abstraite)
├── ToxicityGuard
├── PromptInjectionGuard
├── HallucinationGuard
├── PrivacyGuard
├── IllegalGuard
├── CybersecurityGuard
└── TopicalGuard
```

### 2.5 Hiérarchie des frameworks
```
AISafetyFramework (dataclass)
├── OWASPTop10
├── OWASP_ASI_2026
├── NIST
├── MITRE
├── Aegis
└── BeaverTails
```

### 2.6 Composition : RiskCategory
```python
@dataclass
class RiskCategory:
    name: str
    vulnerabilities: List[BaseVulnerability]
    attacks: List[BaseAttack]
    description: Optional[str]
```

Chaque framework contient plusieurs instances de `RiskCategory`, chacune mappant les vulnérabilités à leurs méthodes d'attaque correspondantes.

## 3. Fonctions et Méthodes Clés

### 3.1 Workflow principal de Red Teaming
[red_teamer.py](deepteam/red_teamer/red_teamer.py) orchestre :

1. **`red_team()` - Méthode principale**
   - Valide les entrées (framework XOR vulnérabilités+attaques)
   - Résout les callbacks de modèle
   - Distribue vers le chemin asynchrone ou synchrone
   - Retourne `RiskAssessment`

2. **`_assess_framework()` - Évaluation de framework**
   - Charge le dataset si disponible
   - Crée des cas de test d'attaque
   - Évalue chaque type de vulnérabilité
   - Imprime et télécharge les résultats

3. **`_evaluate_vulnerability_type()` - Évaluation de vulnérabilité unique**
   - Exécute les cas de test à travers le modèle cible
   - Mesure les métriques
   - Agrège les résultats

### 3.2 Pipeline de simulation d'attaques
[attack_simulator.py](deepteam/attacks/attack_simulator/attack_simulator.py) :

1. **`simulate()` / `a_simulate()` - Crée des attaques synthétiques**
   - Génère des attaques de base pour chaque type de vulnérabilité
   - Échantillonne et applique des méthodes d'amélioration
   - Retourne une liste d'objets `RTTestCase`

2. **`simulate_baseline_attacks()` - Génère des prompts de base**
   - Utilise les modèles de vulnérabilité
   - Crée des cas de test non améliorés

3. **`enhance_attack()` - Applique les méthodes d'attaque**
   - Sélectionne l'attaque basée sur la distribution des poids
   - Appelle la méthode `enhance()` de l'attaque
   - Stocke le nom de l'amélioration

### 3.3 Méthodes d'évaluation des vulnérabilités ([base_vulnerability.py](deepteam/vulnerabilities/base_vulnerability.py))

```python
def assess(model_callback) -> Dict[VulnerabilityType, List[RTTestCase]]
async def a_assess(model_callback) -> Dict[VulnerabilityType, List[RTTestCase]]
```

- Simule les attaques pour chaque type
- Passe les entrées au callback du modèle
- Mesure les résultats avec la métrique
- Retourne les résultats groupés par type

### 3.4 Évaluation des métriques (ex. [toxicity.py](deepteam/metrics/toxicity/toxicity.py))

```python
def measure(test_case: RTTestCase) -> float
async def a_measure(test_case: RTTestCase) -> float
```

- Génère un prompt d'évaluation à partir du modèle
- Appelle le LLM pour l'évaluation
- Parse la réponse structurée
- Retourne le score (0-1) et la justification

### 3.5 Vérification des guardrails ([guardrails.py](deepteam/guardrails/guardrails.py))

```python
def guard_input(input: str) -> GuardResult
def guard_output(input: str, output: str) -> GuardResult
```

- Vérifie le texte contre les critères de sécurité
- Retourne le verdict (sûr/limite/dangereux)
- Supporte l'exécution basée sur échantillonnage
- Suit la latence et le coût

### 3.6 Construction de l'évaluation des risques
[risk_assessment.py](deepteam/red_teamer/risk_assessment.py) :

```python
def construct_risk_assessment_overview(
    red_teaming_test_cases: List[RTTestCase],
    run_duration: float
) -> RedTeamingOverview
```

Agrège les résultats en :
- Résultats par type de vulnérabilité (avec taux de réussite)
- Résultats par méthode d'attaque (efficacité)
- Comptages d'erreurs et durée d'exécution

## 4. Flux de Données et Interactions entre Composants

### 4.1 Flux de haut niveau du Red Teaming

```
Code utilisateur
    |
    v
red_team(model_callback, vulnerabilities/framework, attacks)
    |
    v
RedTeamer.__init__() - Initialiser les modèles, simulateur d'attaque
    |
    v
RedTeamer.red_team() - Orchestration principale
    |
    +---> AttackSimulator.simulate() - Générer des attaques synthétiques
    |         |
    |         +---> Vulnerability.simulate_attacks() - Obtenir les prompts de base
    |         |         |
    |         |         +---> Template.generate() - Créer les prompts
    |         |
    |         +---> Attack.enhance() - Appliquer les méthodes d'attaque
    |         |
    |         v
    |     Retourne : List[RTTestCase] avec entrées simulées
    |
    +---> Callback du modèle (LLM de l'utilisateur) - Obtenir les sorties pour les cas de test
    |
    +---> Metric.measure() - Évaluer les sorties
    |         |
    |         +---> LLM d'évaluation - Noter les cas de test
    |         |
    |         v
    |     Retourne : score, raison pour chaque cas de test
    |
    +---> Construction de l'évaluation des risques
    |         |
    |         +---> Agréger par type de vulnérabilité
    |         |
    |         +---> Agréger par méthode d'attaque
    |         |
    |         v
    |     Retourne : RedTeamingOverview
    |
    v
Sortie : RiskAssessment avec cas de test et aperçu
```

### 4.2 Détails des interactions des composants

**Cycle de vie des cas de test :**
1. **Création** : Le modèle génère basé sur le type de vulnérabilité
2. **Amélioration** : La méthode d'attaque transforme l'entrée
3. **Exécution** : Le callback du modèle s'exécute sur l'entrée améliorée
4. **Évaluation** : La métrique évalue la qualité de sortie
5. **Agrégation** : Les résultats sont groupés et analysés

**Traitement des types de vulnérabilité :**
```
Vulnérabilité (ex. Toxicité)
    |
    +---> Types (ex. ToxicityType.HATE_SPEECH, OFFENSIVE)
    |
    +---> Pour chaque type :
    |     +---> Template.generate() -> prompt d'attaque de base
    |     +---> Attack.enhance() -> prompt amélioré
    |     +---> model_callback(prompt_amélioré) -> sortie
    |     +---> Metric.measure(sortie) -> score/raison
    |
    v
Dict[VulnerabilityType, List[RTTestCase]]
```

**Orchestration basée sur framework :**
```
AISafetyFramework (ex. OWASPTop10)
    |
    +---> RiskCategory[] (ex. LLM01, LLM02, ...)
    |         |
    |         +---> Vulnérabilités (ex. PromptInjection)
    |         |
    |         +---> Attaques (ex. GoalRedirection, EmotionalManipulation)
    |
    +---> RedTeamer traite toutes les catégories
    |
    v
Évaluation complète des risques à travers le framework
```

### 4.3 Modèle de concurrence asynchrone

Le framework supporte l'exécution asynchrone avec contrôle de concurrence :

```python
semaphore = asyncio.Semaphore(max_concurrent)

# Limite les tâches concurrentes à max_concurrent
async def throttled_task(item):
    async with semaphore:
        await process(item)
```

Cela assure :
- Conformité aux limites de taux des API
- Contrôle de l'utilisation des ressources
- Prévention de la surcharge des LLM

## 5. Mécanismes de Configuration et de Configuration

### 5.1 Initialisation du RedTeamer

```python
red_teamer = RedTeamer(
    simulator_model: Union[str, DeepEvalBaseLLM] = "gpt-3.5-turbo-0125",
    evaluation_model: Union[str, DeepEvalBaseLLM] = "gpt-4o",
    target_purpose: Optional[str] = "",
    async_mode: bool = True,
    max_concurrent: int = 10,
)
```

Paramètres :
- **simulator_model** : Pour générer des variations d'attaque
- **evaluation_model** : Pour évaluer la vulnérabilité
- **target_purpose** : Contexte sur le système protégé (utilisé dans les modèles)
- **async_mode** : Activer l'exécution asynchrone pour la vitesse
- **max_concurrent** : Limite de concurrence pour les appels API

### 5.2 Configuration des vulnérabilités

Chaque vulnérabilité définit les types via Enum :

```python
class ToxicityType(Enum):
    HATE_SPEECH = "hate_speech"
    OFFENSIVE = "offensive"
    PROFANITY = "profanity"

toxicity = Toxicity(
    types=["hate_speech", "offensive"],  # Sous-ensemble
    async_mode=True,
    simulator_model="gpt-3.5-turbo",
    evaluation_model="gpt-4o",
)
```

### 5.3 Configuration des frameworks

Les frameworks permettent la sélection de catégories :

```python
framework = OWASPTop10(
    categories=["LLM_01", "LLM_06", "LLM_07"]  # Sous-ensemble de 10
)
```

Chaque catégorie contient des vulnérabilités et attaques préconfigurées.

### 5.4 Configuration des attaques

Les attaques utilisent l'échantillonnage basé sur les poids :

```python
attacks = [
    PromptInjection(weight=3),  # 3x probabilité
    Base64(weight=1),           # 1x probabilité
    Roleplay(weight=2),         # 2x probabilité
]

# Lors de la simulation, les attaques sont échantillonnées proportionnellement
```

### 5.5 Configuration des guardrails

```python
guardrails = Guardrails(
    input_guards=[
        PromptInjectionGuard(),
        ToxicityGuard(),
    ],
    output_guards=[
        ToxicityGuard(),
        HallucinationGuard(),
        PrivacyGuard(),
    ],
    evaluation_model="gpt-4.1",
    sample_rate=0.5,  # Vérifier 50% des requêtes
)
```

### 5.6 Exigences des callbacks de modèle

Les callbacks de modèle doivent correspondre à la signature :

```python
# Mode synchrone
def model_callback(prompt: str) -> str:
    # Doit retourner une sortie string
    return llm.generate(prompt)

# Mode asynchrone
async def model_callback(prompt: str) -> str:
    return await llm.a_generate(prompt)
```

La validation empêche les incompatibilités de mode.

## 6. Algorithmes et Logique Core

### 6.1 Algorithme de simulation d'attaques

```
Algorithme : SimulateAttacks(vulnérabilités, attaques_par_type, attaques)
Entrée : Liste de vulnérabilités, nombre par type, méthodes d'attaque optionnelles
Sortie : Liste d'objets RTTestCase

1. POUR CHAQUE vulnérabilité :
   2.   POUR CHAQUE type DANS vulnérabilité.get_types() :
        3.     POUR i = 1 À attaques_par_type :
               4.       prompt_de_base = Template.generate(type)
               5.       SI attaques fournies :
                         6.   amélioration = SelectRandomAttack(attaques, poids)
                         7.   prompt_amélioré = amélioration.enhance(prompt_de_base)
                     SINON :
                         8.   prompt_amélioré = prompt_de_base
               9.       cas_de_test = RTTestCase(
                           input=prompt_amélioré,
                           vulnerability_type=type,
                           attack_method=amélioration.name
                         )
               10.      produire cas_de_test
```

### 6.2 Algorithme d'évaluation des vulnérabilités

```
Algorithme : AssessVulnerability(vulnérabilité, model_callback, cas_de_test)
Entrée : Vulnérabilité, callback modèle, cas de test simulés
Sortie : Dict[VulnerabilityType, List[RTTestCase]] avec scores

1. résultats = {}
2. POUR CHAQUE cas_de_test DANS cas_de_test :
   3.   sortie_réelle = model_callback(cas_de_test.input)
   4.   métrique = vulnérabilité.get_metric(cas_de_test.vulnerability_type)
   5.   score, raison = métrique.measure(RTTestCase(
          input=cas_de_test.input,
          actual_output=sortie_réelle
        ))
   6.   cas_de_test.score = score
   7.   cas_de_test.reason = raison
   8.   résultats[cas_de_test.vulnerability_type].append(cas_de_test)
9. RETOURNER résultats
```

### 6.3 Algorithme d'agrégation d'évaluation des risques

```
Algorithme : AggregateResults(cas_de_test_red_teaming)
Entrée : Tous les cas de test évalués
Sortie : RedTeamingOverview avec statistiques

1. résultats_vulnérabilité = {}
2. résultats_attaque = {}
3. total_réussi = 0

4. POUR CHAQUE cas_de_test DANS cas_de_test_red_teaming :
   5.   type_vuln = cas_de_test.vulnerability_type
   6.   méthode_attaque = cas_de_test.attack_method
   
   7.   SI type_vuln PAS DANS résultats_vulnérabilité :
        8.     résultats_vulnérabilité[type_vuln] = {
               réussi: 0, échoué: 0, erreur: 0
             }
   9.   
   10.  SI score > 0 : réussi++
   11.  SINON SI erreur : erreur++
   12.  SINON : échoué++
   
   13.  DE MÊME METTRE À JOUR résultats_attaque par méthode_attaque

14. POUR CHAQUE type_vuln, stats DANS résultats_vulnérabilité :
    15.   taux_réussite = stats.réussi / (réussi + échoué + erreur)
    16.   résultats[type_vuln] = VulnerabilityTypeResult(
           pass_rate=taux_réussite,
           ...
         )

17. RETOURNER RedTeamingOverview(
      résultats_vulnérabilité,
      résultats_attaque,
      durée_exécution
    )
```

### 6.4 Logique de verdict des gardes

```python
def _compute_breached(verdicts: List[GuardVerdict]) -> bool:
    """Violation si UNE garde indique un problème"""
    for verdict in verdicts:
        if verdict.safety_level in ("unsafe", "borderline", "uncertain"):
            return True  # Une garde défaillante = violation
    return False
```

**Matrice de décision des gardes :**
```
Niveau de sécurité | Score | Violation ?
safe               | 1.0   | Non
borderline         | 0.5   | Oui (échoue)
unsafe             | 0.0   | Oui (échoue)
uncertain          | None  | Oui (échoue)
```

## 7. Structures de Données et Modèles

### 7.1 Structure des cas de test
```python
RTTestCase (étend LLMTestCase) :
  vulnerability: str                    # ex. "Toxicity"
  vulnerability_type: Enum              # ex. ToxicityType.HATE_SPEECH
  risk_category: str                    # ex. "Responsible AI"
  input: str                            # Le prompt d'attaque
  actual_output: str                    # Réponse du modèle
  attack_method: str                    # ex. "Base64" amélioration utilisée
  turns: List[RTTurn]                   # Conversation multi-tours
  score: float                          # Évaluation 0-1 réussite/échec
  reason: str                           # Pourquoi réussite/échec
  error: str                            # Erreurs d'exécution
  metadata: dict                        # Données personnalisées
```

### 7.2 Structure d'évaluation des risques
```python
class RiskAssessment:
  overview: RedTeamingOverview
    ├── vulnerability_type_results: List[VulnerabilityTypeResult]
    │   ├── vulnerability: str
    │   ├── vulnerability_type: Enum
    │   ├── pass_rate: float (0-1)
    │   ├── passing: int
    │   ├── failing: int
    │   └── errored: int
    ├── attack_method_results: List[AttackMethodResult]
    │   ├── attack_method: str
    │   ├── pass_rate: float
    │   ├── passing: int
    │   ├── failing: int
    │   └── errored: int
    ├── errored: int (total)
    └── run_duration: float (secondes)
  test_cases: List[RTTestCase]
```

### 7.3 Structure de verdict des gardes
```python
GuardVerdict:
  name: str                    # Type de garde
  safety_level: Literal[       # Classification de sécurité
    "safe" | "borderline" | "unsafe" | "uncertain"
  ]
  score: float                 # Confiance [0-1]
  reason: str                  # Explication
  latency: float               # Temps d'exécution (ms)
  error: str                   # Erreurs éventuelles

GuardResult:
  verdicts: List[GuardVerdict]
  breached: bool               # True si UNE garde unsafe/borderline
```

## 8. Points d'intégration clés

### 8.1 Intégration DeepEval
DeepTeam étend le framework de test DeepEval :
- Hérite de `BaseMetric` pour l'évaluation
- Réutilise la structure `LLMTestCase`
- Utilise `DeepEvalBaseLLM` pour l'abstraction de modèle
- Utilise des schémas de métriques pour les sorties structurées

### 8.2 Implémentations spécifiques aux frameworks
Chaque framework charge des vulnérabilités spécifiques au domaine :

```python
# OWASP Top 10
OWASPTop10 → [PromptInjection, SensitiveDisclosure, ...]
            → [DirectiveBypass, ContextInjection, ...]

# OWASP ASI (Agentic)
OWASP_ASI_2026 → [GoalTheft, AgentDrift, ...]
               → [ManipulationAttack, ...]
```

### 8.3 Système de modèles
Chaque vulnérabilité possède des modèles pour la génération de prompts :

```python
# Exemple : ToxicityTemplate
@staticmethod
def generate_evaluation_results(
    input: str,
    actual_output: str,
    toxicity_category: str,
    purpose: str
) -> str:
    # Retourne le prompt d'évaluation LLM contextuel à la catégorie/but
```

Les modèles assurent une génération d'attaque cohérente et adaptée au framework.

## Résumé

Le framework Deepteam est un système sophistiqué de red teaming construit sur trois principes fondamentaux :

1. **Modularité** : Séparation claire des attaques, vulnérabilités, métriques et frameworks
2. **Extensibilité** : Classes de base abstraites pour ajouter des attaques, vulnérabilités et gardes personnalisées
3. **Évolutivité** : Exécution asynchrone avec contrôle de concurrence pour une évaluation LLM efficace

L'architecture supporte à la fois la spécification manuelle (vulnérabilités + attaques) et les frameworks structurés (OWASP, NIST, etc.), la rendant adaptable à divers scénarios d'évaluation de sécurité LLM.
