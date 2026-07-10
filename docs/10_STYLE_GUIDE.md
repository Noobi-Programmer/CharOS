# 10_STYLE_GUIDE.md

> **Purpose:** Define the visual design language, accessibility standards, and component patterns for CharOS.
> This document ensures consistent, high-quality UI that makes Nila feel like a living, trustworthy companion.

---

## 1. Design Philosophy

### 1.1 CharOS Aesthetic Identity

> **Design should express personality, not competition.**

CharOS visual language is designed to support Nila's character while remaining flexible for other characters:

- **Primary emotion**: Warm, inviting, approachable
- **Visual weight**: Light, breathable, non-overwhelming
- **Interaction cue**: Subtle, natural, never aggressive
- **Animation style**: Organic, breathing, expressive

### 1.2 Core Visual Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| ** softness over harshness** | Gentle edges, subtle shadows | Border-radius: 8-12px, box-shadow: gentle | 
| **Breathing over static** | Continuous, natural movement | CSS animations, fluid transitions |
| **Peripheral focus** | Character presence without intrusion | Semi-transparent backgrounds, minimal UI |
| **Promise over complexity** | What Nila can do, not how | Clean card UI, action-focused labels |
| **Privacy by design** | Information hierarchy based on trust | Minimal data display, clear consent |

---

## 2. Color System

### 2.1 Color Palette

```
Primary Colors (Nila's Heart)
────────────────────────────
• Main Pink:    #FF69B4 (Warm, approachable)
• Bright Pink:  #FF85C1 (Playful, energetic)  
• Light Pink:   #FFA3CE (Gentle, soothing)
• Softer Pink:  #FFC1DB (Comforting, soft)
• Whisper Pink: #FFE0ED (Delicate, subtle)

Secondary Colors (Support System)
────────────────────────────────
• Ocean Teal:    #06B6D4 (Calm, focused)
• Sunset Orange: #F97316 (Energetic, warm)
• Forest Green:  #10B981 (Growth, reassurance)
• Violet Purple: #6366F1 (Imaginative, creative)

Neutral Colors (Foundation)
──────────────────────────────
• Background:    #0F172A (Deep, stable)
• Surface:       #1E293B (Elevated, clear)
• Border:        #334155 (Defined, gentle)
• Text Primary:  #F1F5F9 (Clear, readable)
• Text Secondary: #94A3B8 (Supporting, subtle)
```

### 2.2 Color Usage Guidelines

#### Primary Colors (Emotion & Identity)
- **Use for**: Main character interactions, primary actions, emotional emphasis
- **Rules**: Never use for warnings, errors, or critical system messages
- **Accessibility**: Must meet WCAG AAA contrast ratios

#### Secondary Colors (Context & Function)
- **Ocean Teal**: System status, progress, focus mode
- **Sunset Orange**: Warnings, important actions, time-sensitive items
- **Forest Green**: Success, completion, positive feedback
- **Violet Purple**: Special features, new content, experimentation

#### Neutrals (Content & Structure)
- **Background**: Always light mode by default; dark mode as user choice
- **Surface**: Cards, panels, modals with depth
- **Borders**: Subtle dividers, not for emphasis
- **Text**: Hierarchical sizing, clear hierarchy

### 2.3 Color Variations

**High Contrast Mode:**

```css
.hc-mode {
  /* Replace all colors with high contrast variants */
  --primary: #0066CC;
  --secondary: #FF6600;
  --background: #FFFFFF;
  --surface: #F2F2F2;
  --text-primary: #000000;
  --text-secondary: #333333;
}

/* High contrast component overrides */
.hc-mode .speech-bubble {
  border: 2px solid var(--primary);
  background: rgba(255, 255, 255, 0.95);
  color: var(--text-primary);
}
```

**Reduced Motion Override:**

```css
.reduced-motion {
  /* All animations disabled */
  *, *::before, *::after {
    animation: none !important;
    transition: none !important;
    animation-delay: 0 !important;
    animation-duration: 0.01ms !important;
  }
  
  /* Replace with static equivalents */
  .breathing {
    animation: none;
  }
  
  .pulse {
    animation: none;
    opacity: 1;
  }
}
```

---

## 3. Typography

### 3.1 Font Hierarchy

**CharOS uses system fonts for performance and instant loading:**

| Element | Font | Size | Weight | Line Height | Examples |
|---------|------|------|--------|-------------|----------|
| **Display** | System UI | 24px | 600 | 32px | App title, onboarding |
| **Heading** | System UI | 20px | 500 | 28px | Section headers |
| **Body** | System UI | 16px | 400 | 24px | Main content, dialogue |
| **UI** | System UI | 14px | 500 | 20px | Buttons, labels, captions |
| **Caption** | System UI | 12px | 400 | 16px | Metadata, timestamps |
| **Code** | JetBrains Mono/Fira Code | 13px | 400 | 18px | Code snippets, commands |

### 3.2 Typography Scale

**Responsive Typography:**

```scss
// Base font size scales with viewport
html {
  font-size: 16px;
  
  @media (max-width: 480px) {
    font-size: 14px;
  }
  
  @media (min-width: 1920px) {
    font-size: 18px;
  }
}

// Heading scales
h1 {
  font-size: clamp(24px, 4vw, 48px);
  line-height: 1.2;
}

h2 {
  font-size: clamp(20px, 3vw, 36px);
  line-height: 1.3;
}

// Body text responsive
body {
  font-size: clamp(14px, 2vw, 16px);
  line-height: 1.5;
}
```

### 3.3 Font Usage Rules

**Sentence case:**
- **Proper nouns:** Nila, Rila, Character
- **Brand terms:** CharOS (always title case)
- **Technical terms:** API, UI, VRM (all caps)
- **Default:** Sentence case for user-facing text

**No italics, no all-caps except for very short, high-impact text.**

**No decorative fonts unless character-specific.**

---

## 4. Component Patterns

### 4.1 Speech Bubble Component

**Structure:**

```typescript
interface SpeechBubbleProps {
  text: string;
  type: 'say' | 'think' | 'success' | 'error' | 'ask' | 'whisper';
  position: 'left' | 'right';
  isVisible: boolean;
  onDismiss?: () => void;
  actions?: BubbleAction[];
  expression?: string;
  avatar?: CharacterAvatar;
}
```

**Visual Spec:**

```scss
.speech-bubble {
  /* Layout */
  position: absolute;
  max-width: 320px;
  padding: 16px 20px;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  
  /* Content */
  font-size: 14px;
  line-height: 1.5;
  color: var(--text-primary);
  white-space: pre-wrap;
  word-wrap: break-word;
  
  /* Position-specific styles */
  &.bubble-right {
    right: 20px;
    border-bottom-right-radius: 4px;
    &:after {
      right: -8px;
      border-left: 8px solid var(--bubble-background);
    }
  }
  
  &.bubble-left {
    left: 20px;
    border-bottom-left-radius: 4px;
    &:after {
      left: -8px;
      border-right: 8px solid var(--bubble-background);
    }
  }
  
  /* Type-specific colors */
  &.bubble-say {
    background: rgba(255, 105, 180, 0.95);
    color: white;
  }
  
  &.bubble-think {
    background: rgba(100, 116, 139, 0.9);
    color: rgba(255, 255, 255, 0.9);
    font-style: italic;
  }
  
  /* State animations */
  &.entering {
    opacity: 0;
    transform: scale(0.8);
  }
  
  &.entered {
    opacity: 1;
    transform: scale(1);
  }
  
  &.exiting {
    opacity: 0;
    transform: scale(0.9);
  }
}
```

### 4.2 Character Component

**Character display system:**

```typescript
interface CharacterDisplayProps {
  state: CharacterState;
  className?: string;
  onAnimationEnd?: (animation: string) => void;
}

interface CharacterRenderer {
  // 3D rendering
  renderVRM(modelUrl: string, container: HTMLElement): Promise<void>;
  setAnimation(animationName: string, loop?: boolean): Promise<void>;
  setExpression(expressionName: string, weight: number): void;
  
  // State management
  setState(state: CharacterState): Promise<void>;
  getCurrentState(): CharacterState;
  
  // Interaction
  setInteractive(enabled: boolean): void;
}
```

### 4.3 Layout Components

**Overlay system:**

```typescript
interface OverlayContainer {
  // Container properties
  position: 'fixed' | 'absolute';
  width: number;
  height: number;
  zIndex: number;
  
  // Position states
  position: 'left' | 'right' | 'center';
  isVisible: boolean;
  isMinimized: boolean;
  
  // Interaction
  canClickThrough: boolean;
  onResize?(size: {width: number, height: number}): void;
  onPositionChange?(position: 'left' | 'right'): void;
}

interface CharacterDock {
  // Dock behavior
  anchor: 'left' | 'right';
  snapThreshold: number; // 0-1
  
  // Visual styling
  width: number;
  maxWidth: number;
  minWidth: number;
  
  // Interaction
  isResizable: boolean;
  isDraggable: boolean;
}
```

---

## 5. Animation System

### 5.1 Animation Principles

> **Animation serves communication, not decoration.**

**Animation Guidelines:**

- **Purposeful:** Every animation has clear communication value
- **Natural:** Spring physics, easing functions that feel organic
- **Respectful:** Don't override user preferences (reduced motion)
- **Performance:** GPU-accelerated where possible
- **Coordinated:** Multiple animations work together, not compete

### 5.2 Animation Types

**State Transitions:**

- **Entry/Exit**: Appearance/disappearance (fade, slide, scale)
- **State Changes**: Between character states (blend, transition)
- **Micro-reactions**: Immediate feedback (brighter color, subtle movement)
- **Progress**: Duration indication (loading bars, progress rings)

**Animation Speeds:**

```css
/* Animation timing for different contexts */
.animation-fast {
  animation-duration: 0.2s;
}

.animation-normal {
  animation-duration: 0.4s;
}

.animation-slow {
  animation-duration: 0.8s;
}

/* Easing functions */
.easing-spring {
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

.easing-smooth {
  animation-timing-function: cubic-bezier(0.0, 0, 0.2, 1);
}

.easing-bounce {
  animation-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
```

### 5.3 Character Animations

**Nila's animation palette:**

| State | Animation | Duration | Expression |
|-------|-----------|----------|------------|
| **Idle** | `idleBreathing` | 4s loop | neutral, subtle smile |
| **Listening** | `waveform` | 0.5s loop | eyebrows up |
| **Thinking** | `thinkingThink` | 1s | brow furrow, chin to hand |
| **Planning** | `thinkingNotebook` | 1.5s | notebook appear |
| **Working** | `workingLaptop` | 1s loop | typing, laptop glow |
| **Speaking** | `speakingSpeak` | variable sync | viseme-driven |
| **Success** | `celebrationBounce` | 2s | sparkle effect |
| **Error** | `wince` | 0.5s | shake, "X" bubble |

---

## 6. Interaction Patterns

### 6.1 Mouse & Touch

**Consistent interaction targets:**

- **Touch target size**: 44px minimum (iOS accessibility guideline)
- **Hit testing**: 8px padding around interactive elements
- **Touch feedback**: tactile feedback, visual ripples
- **Hover states**: subtle opacity changes, not color shifts

**Gesture patterns:**

- **Tap**: Immediate visual response, state change
- **Swipe**: Quick navigation, confirm/dismiss patterns
- **Pinch**: Zoom/pan for detailed views
- **Long-press**: Context menu, additional actions

### 6.2 Keyboard Navigation

**Tab order:**

1. Character rendering area
2. Speech bubble close buttons
3. Interactive elements in speech bubble
4. Theme/selection toggles
5. Settings panel
6. System shortcuts

**Shortcut patterns:**

```typescript
const keyboardShortcuts = {
  'Mod+Shift+C': 'toggleOverlay',           // Cross-platform Ctrl/Cmd+Shift+C
  'Mod+K': 'focusSearch',                  // Search focus
  'Escape': 'dismissCurrent',             // General dismiss
  'Mod+Shift+P': 'openProfile',            // Profile/settings
  'Mod+Shift+S': 'startRecording',         // Voice recording
};
```

### 6.3 Voice Commands

**Command structure:**

```
Action Pattern: {verb} {object} [with parameters]
Examples:
• "Hello Nila" → say greeting
• "What can you do?" → list skills  
• "Show me file browser" → open file system
• "Fix this code" → execute refactoring skill
• "Remember to call Mom Friday" → store episodic memory
```

---

## 7. Accessibility

### 7.1 WCAG Compliance

**CharOS targets WCAG 2.1 Level AA:**

- **Perceivable**: Text content, images, audio descriptions
- **Operable**: Keyboard navigation, focus management
- **Understandable**: Clear language, consistent navigation
- **Robust**: Semantic HTML, ARIA labels

**Specific Requirements:**

- **Color contrast**: 4.5:1 for normal text, 3:1 for large text
- **Focus indicators**: Visible 2px outline, high contrast
- **Screen reader**: Complete ARIA labels, live regions
- **Touch targets**: Minimum 44px diameter
- **Keyboard navigation**: Full keyboard access, logical order

### 7.2 Screen Reader Support

**Live regions for dynamic content:**

```html
<!-- Speech bubble announcements -->
<div aria-live="polite" aria-atomic="true" class="sr-only">
  <span id="bubble-announcements"></span>
</div>

<!-- Character state announcements -->
<div aria-live="assertive" aria-atomic="true" class="sr-only">
  <span id="character-status-announcements"></span>
</div>

<!-- Skill execution announcements -->
<div aria-live="gentle" aria-atomic="true" class="sr-only">
  <span id="skill-announcements"></span>
</div>
```

**ARIA patterns:**

- **Alert dialog**: Speech bubble with dismiss action
- **Status**: Character state updates
- **Navigation**: Main overlay navigation
- **Button**: Interactive elements

### 7.3 Focus Management

**Focus trapping:**

- **Modal dialogs**: Focus trapped within dialog
- **Speech bubbles**: Focus on close button
- **Character overlay**: Focus on interaction area
- **External applications**: Focus returns on overlay close

**Focus visibility:**

```css
:focus {
  outline: 3px solid var(--primary);
  outline-offset: 2px;
  box-shadow: 0 0 0 5px rgba(255, 105, 180, 0.2);
  position: relative;
}

/* High contrast focus */
.hc-mode :focus {
  outline-width: 4px;
  outline-color: var(--hc-primary);
}
```

---

## 8. Documentation Standards

### 8.1 Documentation Format

**CharOS documentation follows consistent structure:**

```markdown
# [Title]

> **Purpose:** Brief description of document scope

---

## Section 1: Introduction

> Overview and context

## Section 2: Technical Details

> Detailed specifications

### Subsection 2.1: Implementation

> Code examples

### Subsection 2.2: Architecture

> System design

```

**Headers**: Use sentence case for all headers
**Lists**: Use consistent indentation
**Code blocks**: Include file paths in comments
**Diagrams**: Use Mermaid with alt text
**Tables**: Simple 2-column preferred

### 8.2 Code Documentation

**JavaScript/TypeScript comments:**

```typescript
/**
 * Component: SpeechBubble
 * 
 * Displays character dialogue with appropriate styling based on content type.
 * 
 * @param props - Component properties
 * @returns Speech bubble element
 * @example
 * <SpeechBubble 
 *   type="say"
 *   text="Hello! I'm Nila."
 *   position="right"
 * />
 */
function SpeechBubble(props: SpeechBubbleProps): JSX.Element {
  // Implementation
}
```

**Rust documentation:**

```rust
//! Character runtime implementation
//! 
//! Manages VRM avatar, animation states, and speech bubbles.
//! 
//! @module character-runtime

use std::sync::Arc;
use tokio::sync::RwLock;

/// Character runtime interface
pub trait CharacterRuntime {
    /// Initialize character with model
    fn load_model(&mut self, url: &str) -> Result<(), CharacterError>;
    
    /// Play animation by name
    fn play_animation(&mut self, name: string, loop: bool) -> Result<(), CharacterError>;
    
    /// Set facial expression
    fn set_expression(&mut self, name: &str, weight: f32) -> Result<(), CharacterError>;
}
```

---

## 9. Testing Strategy

### 9.1 Component Testing

**React Testing Library patterns:**

```typescript
// Character component test
import { render, screen, fireEvent } from '@testing-library/react';
import { CharacterDisplay } from './CharacterDisplay';

describe('CharacterDisplay', () => {
  it('renders character with correct state', () => {
    render(<CharacterDisplay character={mockIdleState} />);
    expect(screen.getByTestId('character')).toBeInTheDocument();
  });
  
  it('triggers animations on state change', async () => {
    const { rerender } = render(<CharacterDisplay character={mockIdleState} />);
    await act(async () => {
      await characterDisplay.setState(mockThinkingState);
    });
    expect(screen.getByTestId('animation')).toHaveClass('thinking');
  });
});
```

### 9.2 Visual Regression Testing

**Chromatic integration:**

```yaml
# .chromatic.json
autoAcceptChanges:
  - '**/*.md'
  
  only: [
    '**/*.tsx',
    '**/*.css',
  ]
```

**Visual test scenarios:**

- Character states (idle, thinking, speaking, error)
- Speech bubble types (say, think, success, error)
- Theme variations (light, dark, high-contrast)
- Animation transitions
- Responsive states (desktop, tablet, mobile)

---

## 10. Performance Guidelines

### 10.1 Rendering Optimization

**Efficient component patterns:**

- **Memoization**: `React.memo()` for pure components
- **Controlled components**: Minimize re-renders
- **Virtual scrolling**: For long content lists
- **Image lazy loading**: For character assets
- **CSS containment**: Limit layout thrashing

### 10.2 Animation Optimization

**Performance considerations:**

- **GPU acceleration**: `will-change: transform` for animated elements
- **Frame pacing**: Maintain 60fps for smooth animations
- **Playback rate**: Adjust based on system capabilities
- **Complexity management**: LOD (Level of Detail) for distant elements

### 10.3 Bundle Optimization

**Code splitting strategies:**

- **Route-based**: Lazy load page components
- **Component-based**: Tree-shake unused components
- **Shared dependencies**: Common libraries externally cached

---

## 11. Cross-References

| Document | Field | Relationship |
|----------|-------|-------------|
| `docs/00_VISION.md` | Section 5 | Visual design supports vision |
| `docs/01_ARCHITECTURE.md` | Section 7.1 | UI architecture implementation |
| `docs/02_DESIGN_PHILOSOPHY.md` | Section 6 | Design principles applied |
| `docs/03_TERMINOLOGY.md` | All sections | Canonical terminology |
| `docs/04_PROJECT_STRUCTURE.md` | Section 6.2 | Component organization |
| `docs/05_TECH_STACK.md` | Section 2 | React + Tauri implementation |
| `docs/06_UI_GUIDELINES.md` | Current | Self-reference |
| `docs/07_CHARACTER_GUIDELINES.md` | All sections | Character system integration |
| `docs/08_AI_GUIDELINES.md` | Section 7.3 | AI output display patterns |
| `docs/09_ROADMAP_DETAILS.md` | Section 2.3 | Implementation timeline |
| `character/CHARACTER_SPEC.md` | All sections | Character specification |

---

## 12. Open Design Questions

### 12.1. Motion Design System

| Option | Pros | Cons |
|--------|------|------|
| **Custom animation engine** | Full control, unique feel | Complex implementation |
| **Industry-standard libraries** | Mature, well-tested | Less distinctive |
| **Hybrid approach** | Balance of control and simplicity | Integration complexity |

**Status:** Custom animation system with industry-standard building blocks.

### 12.2 Typography Delivery

| Option | Pros | Cons |
|--------|------|------|
| **System fonts** | Performance, reliability | Less customization |
| **Web fonts** | Branding, consistency | Loading overhead |
| **Variable fonts** | Modern features, performance | Browser support |

**Status:** System fonts with variable font fallbacks.

### 12.3 Component Framework

| Option | Pros | Cons |
|--------|------|------|
| **Design tokens** | Scalable theming, theme switching | Requires build tooling |
| **CSS variables only** | Simple, runtime switching | Limited component isolation |
| **Component libraries** | Rich component ecosystem | Version coupling |

**Status:** CSS variables with component tokens.

---

## 13. TODOs for Implementation

- [ ] Define design token system (colors, typography, spacing)
- [ ] Create component library with consistent API
- [ ] Implement animation system with state machine
- [ ] Setup documentation generation pipeline
- [ ] Write visual regression tests
- [ ] Create accessibility testing suite
- [ ] Setup performance monitoring
- [ ] Record ADR for design system
- [ ] Record ADR for animation delivery
- [ ] Record ADR for typography strategy

---

> **Good design is invisible engineering.**
>
> Design should make the complex feel intuitive and the subtle feel meaningful.
>
> Every visual decision should support the goal of making Nila feel like a living, helpful companion.

---

## 14. Character Design Quick Reference

### Primary Colors (Use for character-facing UI)

| Color | RGB | Usage | Emotion |
|-------|-----|-------|---------|
| **#FF69B4** | Warm pink | Main character, primary actions | Friendly, approachable |
| **#FF85C1** | Bright pink | Highlights, energy elements | Playful, energetic |
| **#FFA3CE** | Light pink | Backgrounds, subtle elements | Gentle, soothing |
| **#FFC1DB** | Soft pink | Supporting UI, secondary info | Calm, comforting |

### Typography Quick Reference

- **Display**: 24px Inter/SF Pro Display, Bold
- **Heading**: 20px Inter/SF Pro Text, Medium  
- **Body**: 16px Inter/SF Pro Text, Regular
- **UI**: 14px Inter/SF Pro Text, Medium
- **Caption**: 12px Inter/SF Pro Text, Regular

### Animation Duration Guidelines

| Context | Duration | Easing |
|---------|----------|--------|
| **Micro-interactions** | 0.2s | cubic-bezier(0.4, 0, 0.2, 1) |
| **State transitions** | 0.4s | cubic-bezier(0.0, 0, 0.2, 1) |
| **Progress indicators** | Variable | linear |
| **Celebration effects** | 2s | cubic-bezier(0.68, -0.55, 0.265, 1.55) |

### Component Spacing

```scss
// CharOS Spacing Scale (8px base)
$space-3xs: 2px;  // Minimal space
$space-2xs: 4px;  // Compact space
$space-xs:  8px;  // Small space
$space-sm:  16px; // Medium space
$space-md:  24px; // Large space
$space-lg:  32px; // Extra-large space
$space-xl:  48px; // Massive space
$space-xxl: 64px; // Colossal space
```

---

> **Remember: The interface is the character. Make it lovable.**
>
> *Every design decision should make Nila feel warmer, more capable, and more trustworthy.*

---

## 15. References

- **Material Design Guidelines** - Color, Typography, Interaction
- **Apple Human Interface Guidelines** - Design principles
- **Google Material You** - Material Theme system
- **W3C WCAG 2.1 Success Criteria** - Accessibility standards
- **MDN Web Docs** - Modern web development practices
- **Three.js Documentation** - 3D rendering foundation

---

> **Build with compassion, design with purpose, and ship with pride.**
>
> *The best user interface is the one that lets users focus on what matters — their interaction with the character.*

---

## 16. Completion Checklist

### Design System Complete
- [ ] Color system documented with variants
- [ ] Typography hierarchy established
- [ ] Component patterns defined
- [ ] Animation system designed
- [ ] Accessibility guidelines written
- [ ] Performance optimization strategies
- [ ] Cross-browser compatibility
- [ ] Responsive design rules
- [ ] Documentation standards
- [ ] Testing strategy
- [ ] Extension points
- [ ] Open design questions
- [ ] Implementation TODOs

> **This design guide will evolve as the platform matures, but will always serve the same purpose:**
>
> **To make CharOS feel like a thoughtful, capable, and delightful desktop companion.**

---

*Last updated: June 30, 2026*
*Version: 1.0.0*