# Nicola Fiorillo

Simple static home page.

## Philosophy: AI as the Static Site Generator

This site explores a minimalist approach to web development where Artificial Intelligence replaces the traditional Static Site Generator (SSG).

### The "No-Engine" Approach
Instead of relying on frameworks like Jekyll, Hugo, or Next.js, this site consists of pure, hand-crafted (via AI) HTML files and Tailwind CSS. There is no build step, no `npm install`, and no configuration files.

When a modification is needed—such as adding a new "Today I Learned" (TIL) entry—I simply ask the AI to "add a new entry about X." The AI understands the context, duplicates the necessary structure, applies the correct Tailwind classes, and updates the links. The AI acts as the "compiler," maintaining consistency across pages without the overhead of a software abstraction layer.

### Benefits
*   **Zero Dependencies**: The site will never break due to a deprecated package or a version mismatch. It is future-proof by design.
*   **Maximum Performance**: The output is raw HTML. Browsers render this instantly without needing to hydrate a JavaScript framework or execute complex client-side logic.
*   **Simplicity**: The codebase is completely transparent. There is no "magic" happening behind the scenes—what you see in the file is exactly what gets served to the browser.
*   **Rapid Iteration**: Using Tailwind via CDN allows for immediate visual feedback and styling without a compilation step.

### Potential Drawbacks
*   **Scalability**: Without templates, changing a shared element (like a header) across hundreds of pages requires updating every file individually, although AI agents can automate this bulk editing.
*   **Consistency**: Relying on AI to maintain structure requires strict context management. A "drift" in style is possible over long periods if the context window isn't provided correctly.
*   **Missing Automations**: Convenient features like automatic image optimization, asset hashing for cache busting, or automated testing are not available out-of-the-box and must be handled manually or via specific AI tasks.

