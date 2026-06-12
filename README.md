Common detectable layers:

Browser automation flags

navigator.webdriver is explicitly defined to indicate that a browser is controlled by automation. MDN describes it as a read-only property showing whether the user agent is controlled by automation, and the WebDriver spec defines WebDriver as a remote-control interface for browsers.

Some CDP-only setups may avoid this particular flag, but many real automation stacks still leak other inconsistencies.

WebGL / GPU fingerprint

Yes, a site can query WebGL characteristics. The WEBGL_debug_renderer_info extension exposes unmasked GPU vendor and renderer strings, subject to browser/privacy settings.

So values like SwiftShader, unusual virtual GPUs, cloud GPU strings, or renderer/OS/header mismatches can contribute to a bot score.

JavaScript/browser fingerprint consistency

Sites can collect screen size, device memory, hardware concurrency, fonts, audio/canvas/WebGL output, permissions behavior, plugins, MIME types, timezone, language, touch support, media codecs, error-stack quirks, extension artifacts, and prototype/function-shape anomalies. The strongest signal is often not any one attribute, but inconsistency: for example, “Windows Chrome” headers with Linux-like fonts, a datacenter IP, odd GPU, no realistic interaction history, and non-human timing.

Network/server-side fingerprint

Even with real headed Chrome, servers and bot-management vendors can score IP reputation, cookie history, request velocity, TLS/HTTP fingerprints, header ordering, challenge responses, and session continuity. Cloudflare says its bot detection uses multiple engines including machine learning and behavioral analysis; Akamai advertises behavior analysis and browser fingerprinting; DataDome describes TLS, browser, behavioral, and reputation-based detection.

Behavioral fingerprint

A multimodal AI agent can still be distinguishable by how it scrolls, clicks, types, pauses, corrects mistakes, navigates, and reacts to dynamic UI. A recent AI-agent fingerprinting study found that browser fingerprints alone may be limited when agents share browser environments, but behavioral fingerprints were distinctive for separating AI browsing agents from humans and from one another. That is a preprint, but it matches how modern bot systems are moving.

So the practical answer is:

A headed, multimodal Chrome agent is less obvious than old headless Selenium, but it is not invisible. Sites may not “detect CDP” directly in a single clean flag, but they can detect automation through a combination of browser API fingerprints, WebGL/GPU strings, automation side effects, network fingerprints, cookies, IP reputation, and behavior.

For legitimate use, the safest approach is not to try to bypass bot detection. Use official APIs, robots.txt-compliant crawling, explicit user-agent identification where appropriate, rate limits, allowlisting/partnership routes, and test accounts or staging environments. If you own the site, instrument your own pages to log the major classes of signals above and compare your agent traffic against real user baselines.
