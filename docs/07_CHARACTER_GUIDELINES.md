# 07_CHARACTER_GUIDELINES.md

> **Purpose:** Define the character system architecture, specification, and extension points for CharOS characters.
> This document governs all character-related design decisions and implementation patterns.

---

## 1. Character System Philosophy

### 1.1 Character-First Architecture

> **"The character is the product. The platform is the infrastructure that enables the character to express itself."**

The character is **not just an avatar**—it is a complete persona with:
- **Appearance** (visual form, style, theme)
- **Voice** (speech patterns, tone, personality)
- **Personality** (behavioral traits, interaction style)
- **Expressions** (emotional range, animation states)
- **Theme** (aesthetic, color scheme, atmosphere)

### 1.2 Platform vs. Character Separation

```
┌─────────────────────────────────────────────────────────────┐
│                   CHAROS PLATFORM                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────┐ ┌────────────────────┐ ┌─────────────────┐ │
│  │   PLANNER      │ │ MEMORY SYSTEM    │ │   UI            │ │
│  │               │ │                   │ │               │ │
│  └──────┬─────────┘ └──────┬─────────────┘ └──────┬─────────┘ │
└─────────┼─────────────────┼─────────────────┼───────────┘
          │                 │                 │
    ┌─────┼─────┐    ┌─────┼─────┐    ┌─────┼─────┐
    │ CHARACTER │    │ SKILLS │    │ MODELS │    │ TOOLS │
    │ SYSTEM    │    │ SYSTEM │    │ SYSTEM │    │ SYSTEM │
    └───────────┘    └─────────┘    └─────────┘    └─────────┘
```

**Key Principle:** The platform owns the infrastructure (plannning, memory, UI, communication), while the character owns the persona experience.

### 1.3 Character as Interface

```typescript
// Character as interface to platform
interface CharacterProvider {
  readonly id: string;                     // Unique identifier
  readonly metadata: CharacterMetadata;    // Static character info
  
  // Runtime behaviors
  getPersonality(): Personality;
  getExpressions(): ExpressionSet;
  getVoice(): VoiceConfiguration;
  getTheme(): Theme;
  
  // State management
  setState(state: CharacterState): Promise<void>;
  triggerAnimation(name: string): Promise<void>;
  showBubble(content: BubbleContent): Promise<void>;
  
  // Event handling
  onUserInteraction(type: InteractionType, data: any): Promise<void>;
}
```

---

## 2. Character Specification System

### 2.1 Character Data Structure

**Core Character Definition:**

```json
{
  "id": "nila-v2",
  "name": "Nila",
  "version": "2.0.0",
  "author": "CharOS Team",
  
  "appearance": {
    "vrmUrl": "assets/characters/nila.vrm",
    "size": {"width": 320, "height": 480},
    "animations": [
      {"name": "idle", "file": "animations/idle.vrm"}, 
      {"name": "thinking", "file": "animations/thinking.glb"}
    ],
    "expressions": ["neutral", "happy", "thinking", "sleepy"]
  },
  
  "personality": {
    "traits": ["curious", "playful", "helpful"],
    "speakingStyle": "casual, uses emoji, occasional puns",
    "greetingPhrases": ["Hi!", "Hello there!", "What's up?"],
    "responsePatterns": {
      "helpful": 0.7,
      "humorous": 0.3,
      "empathetic": 0.5
    }
  },
  
  "voice": {
    "engine": "piper",
    "model": "en-us-lessac",
    "speed": 1.0,
    "pitch": 1.0,
    "emotionMap": {
      "happy": "cheerful",
      "angry": "stern", 
      "sad": "soft"
    }
  },
  
  "theme": {
    "colors": {
      "primary": "#FF69B4",
      "secondary": "#FF85C1",
      "background": "#FFF0F5"
    },
    "style": "cute, friendly, modern",
    "glassmorphism": true,
    "animationIntensity": "medium"
  },
  
  "ui": {
    "speechBubble": {
      "position": "right",
      "shape": "rounded",
      "padding": "md",
      "maxWidth": 320
    },
    "animationSettings": {
      "duration": "normal",
      "easing": "spring",
      "microExpressions": true
    }
  }
}
```

### 2.2 Character Metadata

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `id` | string | Unique character identifier | ✅ |
| `name` | string | Display name | ✅ |
| `version` | string | Character format version | ✅ |
| `author` | string | Character author/creator | ✅ |
| `appearance` | object | Visual configuration | ✅ |
| `personality` | object | Behavioral traits | ✅ |
| `voice` | object | Speech characteristics | ✅ |
| `theme` | object | Visual theme | ✅ |
| `ui` | object | UI customization | ❌ (uses platform defaults) |

### 2.3 Character Pack Format

**Portable Character Bundle (.charos):**

```text
nila-v2.charos/
├── manifest.json              # Character metadata
├── assets/                    # All character assets
│   ├── character.vrm          # VRM model file
│   ├── animations/           # Animation files
│   ├── expressions/          # Expression definitions
│   ├── voice/                # Voice configuration
│   └── themes/               # Theme files
├── personality.json          # Personality definition
├── readMe.md                 # User instructions
└── preview.png              # Thumbnail image
```

---

## 3. Personality System

### 3.1 Personality Traits

**Core Trait Categories:**

| Category | Traits | Description |
|----------|--------|-------------|
| **Social** | extroverted, introverted, friendly, stern | First-impression behaviors |
| **Cognitive** | curious, analytical, creative, pragmatic | Problem-solving approaches |
| **Emotional** | empathetic, humorous, serious, playful | Emotional expression |
| **Work** | meticulous, relaxed, deadline-driven, thorough | Professional habits |

### 3.2 Personality Expression Mapping

```typescript
interface Personality {
  readonly traits: PersonalityTrait[];
  readonly responsePatterns: ResponsePattern[];
  readonly conversationStyles: ConversationStyle[];
  
  // Trait-based behavior analysis
  getTraitIntensity(trait: PersonalityTrait): number;
  matchesContextualContext(context: InteractionContext): boolean;
  
  // Adaptive behavior
  adaptToUserPersonality(userProfile: UserProfile): Personality;
  adjustForConversationPhase(phase: ConversationPhase): Personality;
}

interface PersonalityTrait {
  id: string;
  name: string;
  category: TraitCategory;
  intensity: number; // 0-1
  description: string;
}
```

### 3.3 Conversation Dynamics

**Personality-Influenced Response Patterns:**

| Situation | Basic Response | Curious Personality | Humorous Personality | Analytical Personality |
|-----------|----------------|--------------------|---------------------|------------------------|
| Greeting | "Hello!" | "Wow, nice to meet you! What brings you here?" | "Hey there! It's a beautiful day, isn't it?" | "Hello. I'm Nila, your assistant. How can I be of service?" |
| Error | "I'm sorry, I couldn't do that." | "Oops! My bad. Let me try again." | "Well, that's interesting... Let's try something else." | "Error detected. Root cause analysis required." |
| Celebration | "Good job!" | "Excellent! You did it! What's your secret?" | "Great achievement! Celebrate with me!" | "Task completed successfully. Metrics: {}" |

---

## 4. Expression System

### 4.1 Expression Categories

| Category | Sub-expressions | Use Cases |
|----------|----------------|-----------|
| **Facial** | smile, frown, surprise, blink | Emotional states, reactions |
| **Body** | pose, gesture, movement, posture | Activity, intent, mood |
| **Combined** | thinking_brows, smile_eyes, relaxed | Complex states like thinking |

### 4.2 Expression Management

**Expression Evaluation:**

```typescript
class ExpressionEngine {
  private currentExpressions: Map<string, number> = new Map();
  private expressionRules: ExpressionRule[];
  
  // Dynamic expression blending
  updateExpression(expressionName: string, weight: number): void;
  blendExpressions(baseExpression: string, modifierExpression: string, weight: number): string;
  
  // Context-aware expression selection
  getExpressionForContext(context: InteractionContext): ExpressionSet;
  getNearbyExpressions(expressionName: string): string[];
  
  // Animation integration
  applyExpressionToAnimation(animationClip: AnimationClip, expressions: ExpressionSet): void;
}
```

### 4.3 Micro-Expression System

**Fine-grained Expressions (100-200ms duration):**

- **Attention**: Chin raise, eyebrows up
- **Surprise**: GASP, widened eyes
- **Confusion**: Head scratch, eyebrow raise
- **Approval**: Nod, smile, head tilt
- **Approval**: Slight smile, subtle blink

---

## 5. Voice and Speech System

### 5.1 Voice Configuration

**Voice Parameters:**

```json
{
  "engine": "piper",
  "model": "en-us-lessac",
  "speed": 1.0,
  "pitch": 1.0,
  "volume": 1.0,
  "voiceMap": {
    "happy": "cheerful, bright",
    "sad": "soft, gentle",
    "angry": "deep, firm",
    "thinking": "thoughtful, calm",
    "surprised": "startled, sharp"
  },
  "speechRate": {
    "normal": 1.0,
    "slow": 0.8,
    "fast": 1.2,
    "enthusiastic": 1.3
  },
  "prosody": {
    "emphasis": "natural",
    "intonation": "varied",
    "pausing": "appropriate"
  }
}
```

### 5.2 Voice-to-Text Processing

**Handy-Parakeet Integration:**

> **Real-time speech recognition with context awareness**

```typescript
interface SpeechProcessor {
  readonly provider: SpeechProvider;
  readonly engine: SpeechEngine;
  
  // Audio processing
  startStream(audioSource: MediaStream): Promise<void>;
  stopStream(): Promise<void>;
  
  // Recognition
  processAudio(audioChunk: Float32Array): Promise<TranscriptionSegment>;
  onTranscript(callback: (transcript: Transcription) => void): void;
  
  // Context integration
  setContext(context: SpeechContext): void;
  getContextAwareTranscription(): Promise<TranscriptionContext>;
}
```

### 5.3 Text-to-Speech System

**Emotion-aware synthesis:**

```typescript
class EmotionAwareTTS {
  private voiceConfig: VoiceConfig;
  private emotionMap: EmotionVoiceMap;
  
  async synthesize(text: string, emotion: Emotion, options?: TTSSynthesisOptions): Promise<AudioBuffer> {
    // Select voice configuration based on emotion
    const config = this.emotionMap.getVoiceForEmotion(emotion);
    const adjustedSpeed = this.adjustSpeedForEmotion(config.speed, emotion);
    
    // Generate speech
    return this.ttsEngine.synthesize(text, {
      voice: config,
      speed: adjustedSpeed,
      pitch: config.pitch,
      volume: config.volume
    });
  }
  
  // Emotion-specific tone mapping
  private adjustSpeedForEmotion(baseSpeed: number, emotion: Emotion): number {
    const emotionModifiers: Record<Emotion, number> = {
      'happy': 1.1,
      'sad': 0.9,
      'angry': 1.2,
      'surprised': 1.3,
      'thinking': 1.0
    };
    return baseSpeed * (emotionModifiers[emotion] || 1.0);
  }
}
```

---

## 6. Theme System

### 6.1 Theme Structure

**Hierarchical Theme System:**

```
Platform Theme (default)
├── Base Theme (Light/Dark)
│   ├── Nila Theme (default)
│   │   ├── Primary colors
│   │   ├── Animations
│   │   └── Micro-expressions
│   └── Custom Themes
│       ├── Ocean Theme
│       ├── Forest Theme
│       └── Minimal Theme
└── Character Theme (overrides)
    ├── Colors
    ├── Fonts
    └── Animation styles
```

### 6.2 Theme Variables

**CSS Custom Properties:**

```css
:root {
  /* Platform defaults */
  --charos-primary: #ff69b4;
  --charos-secondary: #ff85c1;
  --charos-background: #fff0f5;
  --charos-surface: #ffffff;
  --charos-text-primary: #333333;
  --charos-text-secondary: #666666;
}

.character-theme {
  --charos-primary: /* Character override */;
  --charos-secondary: /* Character override */;
  --charos-background: /* Character override */;
}
```

### 6.3 Dynamic Theme Adaptation

**Context-aware theming:**

- **Time-based**: Different themes for morning, afternoon, evening
- **Activity-based**: Specialized themes for coding, reading, gaming
- **User preference**: Respect user-selected themes and reduce motion
- **Accessibility**: High-contrast and color-blind safe variants

---

## 7. Character-Pack System

### 7.1 Pack Architecture

**Organized Character Ecosystem:**

```textnpack-repository/
├── characters/                  # All available characters
│   ├── nila/                   # Character pack directory
│   │   ├── manifest.json       # Pack metadata
│   │   ├── assets/             # All pack assets
│   │   └── README.md          # Pack documentation
│   ├── mia/                   # Another character
│   └── [other characters]...
├── themes/                    # Theme pack system
│   ├── nila-default/           # Nila's default theme
│   ├── ocean-theme/           # Water-themed design
│   └── minimal-theme/         # Clean design
├── voices/                    # Voice pack system
│   ├── en-us-lessac/          # English voice
│   ├── ja-jp-pope/           # Japanese voice
│   └── custom-urban/          # User-created voice
├── plugins/                    # Third-party extensions
│   ├── skill-pack/           # Skill extensions
│   ├── character-pack/        # Character extensions
│   └── ui-mod/              # UI modifications
├── examples/                  # Usage examples
│   ├── quick-start/          # Getting started examples
│   ├── workflows/            # Common workflows
│   └── tutorials/            # Video/tutorials documentation
└── documentation/             # Pack documentation
    ├── formats.md            # Pack format specification
    ├── validation.md         # Pack validation guide
    └── submission.md         # Pack contribution guide
```

### 7.2 Pack Manifest

**Pack metadata structure:**

```json
{
  "id": "nila-community-v1",
  "name": "Nila Community Edition",
  "version": "1.0.0",
  "author": "CharOS Community",
  "description": "Community-created Nila variant with new animations",
  "type": "character",
  "compatibility": {
    "platform": "^1.0.0",
    "characters": ["nila"],
    "dependencies": []
  },
  "assets": {
    "vrm": "assets/nila-community.vrm",
    "animations": "assets/animations/",
    "expressions": "assets/expressions/"
  },
  "features": ["new_animations", "custom_voice", "personality_mod"],
  "downloadCount": 1500,
  "rating": 4.8,
  "tags": ["anime", "kyoko", "energetic"]
}
```

### 7.3 Pack Validation

**Pack integrity checks:**

- **Format validation**: Manifest structure and required fields
- **Asset completeness**: All referenced files exist and are readable
- **Character compatibility**: Matches target character schema
- **Performance**: Asset sizes and loading times
- **Legal**: Attribution and licensing compliance

---

## 8. Character Runtime System

### 8.1 Runtime Architecture

**Character execution stack:**

```
┌─────────────────────────────────────────────────────────┐
│                    CHARACTER RUNTIME                    │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────────┐ ┌────────────────────┐ ┌─────────────────┐ │
│  │ CHARACTER      │ │  ANIMATION       │ │  SPEECH         │ │
│  │   ENGINE       │ │   CONTROLLER    │ │   SYSTEM       │ │
│  │                 │ │                   │ │               │ │
│  └──────┬─────────┘ └──────┬─────────────┘ └──────┬─────────┘ │
└─────────┼─────────────────┼─────────────────┼───────────┘
          │                 │                 │
    ┌─────┼─────┐    ┌─────┼─────┐    ┌─────┼─────┐
    │UI   │CORE │    │AUDIO │    │NETWORK │    │SYSTEM │
    │REPL │API │    │PROCESSING│   │COMM     │    │MANAGE │
    └─────┴─────┘    └─────┼─────┘    └─────┼─────┘    └─────┴─────┘
```

### 8.2 Core Character Engine

```typescript
class CharacterEngine {
  private provider: CharacterProvider;
  private animationController: AnimationController;
  private speechSystem: SpeechSystem;
  
  // Core lifecycle
  async initialize(provider: CharacterProvider): Promise<void>;
  async start(): Promise<void>;
  async stop(): Promise<void>;
  async dispose(): Promise<void>;
  
  // State management
  async setState(state: CharacterState): Promise<void>;
  getCurrentState(): CharacterState;
  
  // Interaction handling
  async handleUserInput(input: UserInput): Promise<void>;
  registerEventHandler(type: EventType, handler: EventHandler): void;
  
  // Communication
  emit(event: CharacterEvent): void;
  on(event: EventType, handler: EventHandler): Subscription;
}
```

### 8.3 Animation Controller

**State machine for character animations:**

```typescript
interface AnimationController {
  // State management
  transitionTo(state: CharacterState): Promise<void>;
  getCurrentState(): CharacterState;
  
  // Animation control
  playAnimation(name: string, options?: AnimationOptions): Promise<void>;
  stopAnimation(name: string): void;
  pauseAnimation(): void;
  resumeAnimation(): void;
  
  // Expression control
  setExpression(name: string, weight: number): void;
  blendExpressions(base: string, modifier: string, weight: number): void;
  
  // Timing and synchronization
  setAnimationSpeed(speed: number): void;
  scheduleNextAnimation(): Promise<void>;
}
```

---

## 9. Interaction Patterns

### 9.1 User Interaction Modeling

**Interaction schema:**

```typescript
interface UserInteraction {
  id: string;
  timestamp: number;
  type: InteractionType;
  source: InteractionSource;
  
  // Content
  payload: any;
  context: InteractionContext;
  
  // Metadata
  metadata: InteractionMetadata;
  processingTime: number;
  responseRequired: boolean;
}

enum InteractionType {
  'click', 'tap', 'gesture', 'voice', 
  'typing', 'hotkey', 'menu', 'hover',
  'drag', 'scroll', 'focus', 'blur'
}
```

### 9.2 Conversation Flow Control

**Adaptive conversation management:**

```typescript
class ConversationManager {
  private conversationState: ConversationState;
  private userProfile: UserProfile;
  private contextHistory: InteractionContext[];
  
  // Turn management
  async processInput(input: UserInput): Promise<AssistantResponse>;
  generateResponse(input: UserInput, context: ConversationContext): Promise<AssistantResponse>;
  
  // Context building
  buildConversationContext(): ConversationContext;
  updateUserModel(input: UserInput, response: AssistantResponse): void;
  
  // Flow control
  determineResponseStrategy(context: ConversationContext): ResponseStrategy;
  needsClarification(response: AssistantResponse): boolean;
}
```

---

## 10. Extension Points

### 10.1 Character Extensions

**Plugin-style extensions:**

```typescript
// Character enhancement plugins
interface CharacterExtension {
  readonly id: string;
  readonly name: string;
  readonly version: string;
  
  // Extension points
  canActivate(context: ExtensionContext): boolean;
  activate(): Promise<void>;
  deactivate(): Promise<void>;
  
  // Override behavior
  modifyBehavior(behavior: CharacterBehavior): CharacterBehavior;
  addExpression(name: string, definition: ExpressionDefinition): void;
  addAnimation(name: string, asset: AnimationAsset): void;
}
```

### 10.2 Custom Personality System

**Modularity for custom personality:**

```typescript
interface CustomPersonality {
  id: string;
  name: string;
  author: string;
  
  // Personality definition
  traits: Record<PersonalityTrait, number>;
  responseRules: ResponseRule[];
  conversationPatterns: ConversationPattern[];
  
  // Activation conditions
  applicableContexts: InteractionContextType[];
  requiredCapabilities: string[];
  
  // Versioning
  version: string;
  dependencies: string[];
}
```

---

## 11. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 4.2 | Character ecosystem overview |
| `docs/01_ARCHITECTURE.md` | Section 5.1 | Character subsystem placement |
| `docs/02_DESIGN_PHILOSOPHY.md` | All sections | Design principles for characters |
| `docs/03_TERMINOLOGY.md` | Character section | Canonical character terms |
| `docs/04_PROJECT_STRUCTURE.md` | Section 7.1 | Character directory structure |
| `docs/05_TECH_STACK.md` | Character section | Technology for character runtime |
| `docs/06_UI_GUIDELINES.md` | All sections | UI integration with characters |
| `docs/08_AI_GUIDELINES.md` | Section 7.3 | Character-AI integration patterns |
| `docs/09_ROADMAP_DETAILS.md` | Section 2.3 | Character implementation timeline |
| `character/CHARACTER_SPEC.md` | All sections | Detailed character specification |
| `character/ANIMATIONS.md` | All sections | Animation system details |
| `character/VOICE.md` | All sections | Voice synthesis details |
| `character/THEMES.md` | All sections | Theme system details |

---

## 12. Open Design Questions

### 12.1 Animation Delivery System

| Option | Pros | Cons |
|--------|------|------|
| **VRM Runtime (Three.js)** | Industry standard, extensive tooling | Higher memory usage, JavaScript overhead |
| **WebGPU Native** | Best performance, direct hardware access | Emerging ecosystem, steeper learning |
| **Custom Engine (Babylon.js)** | Good balance, existing documentation | Less community support |
| **Native Plugins (Electron)** | Best performance, stable APIs | Platform fragmentation |

**Status:** VRM Runtime with WebGL optimization for MVP.

### 12.2 Voice Engine Architecture

| Option | Pros | Cons |
|--------|------|------|
| **Handy-Parakeet (local)** | Privacy, performance, offline | Limited language support |
| **System TTS + Enhancement** | Platform integrated, no setup | Limited naturalness |
| **Cloud-based + Fallback** | Rich voices, continuous improvement | Privacy concerns, requires internet |
| **Hybrid approach** | Best of all worlds | Complex implementation |

**Status:** Hybrid with local first, cloud fallback for backup.

### 12.3 Character Intelligence Layer

| Option | Pros | Cons |
|--------|------|------|
| **Rule-based system** | Deterministic, debuggable, fast | Limited complexity |
| **Simple ML models** | Adaptable, learns patterns | Requires training data |
| **External LLM integration** | Maximum capability | Privacy, cost, latency |
| **Mixed approach** | Scalable, maintainable | Integration complexity |

**Status:** Rule-based core with ML enhancements for complex tasks.

---

## 13. TODOs for Implementation

- [ ] Define CharacterMetadata interface with validation
- [ ] Implement CharacterPackFormat with manifest schema
- [ ] Create CharacterEngine with runtime lifecycle
- [ ] Build AnimationController with state machine
- [ ] Set up SpeechSystem with emotion-aware TTS
- [ ] Implement personality trait system
- [ ] Create conversation flow manager
- [ ] Build character extension plugin system
- [ ] Set up custom personality loader
- [ ] Write integration tests for character runtime
- [ ] Record ADR for character format
- [ ] Record ADR for animation system
- [ ] Record ADR for voice synthesis

---

> **The character is not just what we see, but how we feel. A good character makes the AI feel like a friend, not a tool.**
>
> *Character system defines the boundary between technology and personality. Get it right, and everything else becomes meaningful.*