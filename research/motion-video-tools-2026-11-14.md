# Motion & video tools review (2026-11-14)

Source: [@0xMovez tweet](https://x.com/0xmovez/status/2121312707588583455), "20 GitHub repos for a Fable 5.2 motion studio" (posted 2026-11-13). Analyzed for Fuse's shift to ad, analytics and media-management software for businesses.

## Worth adding

| Tool | Link | Fuse use |
|---|---|---|
| Remotion | https://github.com/remotion-dev/remotion | React-coded video. Auto-render a short promo video per business from their photos, deal, and plan card. **Check the license**: free for very small teams, larger companies pay. |
| Remotion Skills | https://github.com/remotion-dev/skills | Official skills so Claude Code can write Remotion compositions. |
| WhisperX | https://github.com/m-bain/whisperX | Word-level captions on business videos. Most social video plays muted. |
| Lottie (lottie-react-native in app) | https://github.com/airbnb/lottie-web | Light vector animation for match celebration, loading, onboarding. |
| motion-design-skill (LottieFiles) | https://github.com/LottieFiles/motion-design-skill | Claude skill for timing, easing, Disney's 12 principles. Swipe/match polish. |
| Motion ai-kit | https://github.com/motiondivision/ai-kit | CSS springs and motion audits for agents. |
| FFmpeg | https://github.com/FFmpeg/FFmpeg | Encoding in a Function if we render ourselves (Mux covers hosting; see business-tools report). |

## Maybe (only if ads get music)

- librosa (https://github.com/librosa/librosa) syncs cuts to the beat.
- Openverse API (https://api.openverse.org/v1/) has free sound effects and music. Each track has its own Creative Commons license, so check its attribution and commercial-use terms.

## Skip

Three.js, Blender agents (scenario-labs/blender-plugin), react-three-rapier, react-postprocessing, three.quarks, Manim, hairline, anatomy, and anatomy-reveal are 3D or explainer tools that don't fit. GSAP and Tweakpane help only if we build a flashy landing page.

## Claude Code prompt (run from C:\Users\User\fuse)

```
Evaluate adding motion/video tools to Fuse for business ad videos and app polish. Rules: ASK before changing any app code, package.json, Firebase config, or committing. Any API key lives only in Windows user env vars, referenced as ${VAR} in .mcp.json, never literal.

1. Skills (install, no app changes): in Claude Code run `/plugin marketplace add remotion-dev/skills` (or follow its README if it installs differently), and install LottieFiles/motion-design-skill and motiondivision/ai-kit per their READMEs. Show me what commands/skills each adds.
2. Remotion spike (plan only, no install yet): read https://github.com/remotion-dev/remotion and its license page. Propose a separate folder (e.g. C:\Users\User\fuse\ad-video) with a Remotion composition "BusinessPromo" that takes props {businessName, photos[], dealText, planCardTitle, brandColor} and renders a 15s 9:16 MP4. Include where captions from WhisperX would go and how rendering would run (Remotion Lambda vs Cloud Run vs a Firebase Function with FFmpeg). Tell me the license cost for a 2-person company.
3. App polish (plan only): check whether lottie-react-native works with our Expo SDK, and list 3 spots (match screen, loading, onboarding) where a Lottie animation would help. Show the package.json change but don't apply it.
Stop after the plans and wait for my OK.
```
