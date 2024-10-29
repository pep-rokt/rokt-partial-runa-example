# ROKT "Partial RUNA" Integration Example

This repo is an example Rokt integration that is partially served via RUNA. To demo, simply run `live-server` at the project root.

![Rakuten _ RUNA](https://github.com/user-attachments/assets/51868add-21b6-4910-b463-257232017c53)

Breaking down the example...
- The Rokt SDK is loaded at the top-level via the following script:

  ```html
  <script type="module">
    window.RoktLauncherScriptPromise = new Promise((resolve, reject) => {
      const target = document.head || document.body;
      const script = document.createElement("script");
      script.type = "text/javascript";
      script.src = "https://apps.rokt.com/wsdk/integrations/launcher.js";
      script.fetchpriority = "high";
      script.crossOrigin = "anonymous";
      script.async = true;
      script.id = "rokt-launcher";
  
      script.addEventListener('load', () => resolve());
      script.addEventListener('error', (error) => reject(error));
  
      target.appendChild(script);
    });
  </script>
  ```
- On execution, Rokt SDK scaffolds a few iframes necessary for the integration, including the `rokt-controller-frame` and `Rokt placement` iframes. The `Rokt placement` iframe is responsible for rendering Rokt offers 
- A second Rokt script is loaded via RUNA, which accesses the top-level Rokt instance to integrate data attributes and fetch eligible Rokt offers:
  ```html
  <script type="module">
    // Use top-level Rokt launcher to fetch offers (assuming top-level app and iframes share same-domain)
    // Rokt Layout will render at the top-level where the SDK where the rokt-contorller-frame exists
    await window.top.RoktLauncherScriptPromise;
    const launcher = await window.top.Rokt.createLauncher({
      accountId: "3051387316918867566", // test account
      sandbox: true,
    });
    
    // attributes should be correctly mapped to values available through RUNA
    await launcher.selectPlacements({
      attributes: {
        email: "customer@gmail.com",
        firstname: "John",
        lastname: "Doe",
        confirmationref: "12345",
      },
      identifier: 'confirm'
    });
  </script>
  ```
- Since the Rokt instance lives in the top-level app, the Rokt Overlay will also get rendered at the top-level of the application (outside of any RUNA iframe)
- Note that the RUNA iframes in this example were hardcoded (and not injected via the `rdntag.display()` call). This is to override the [sandbox Rokt integration](https://storage.googleapis.com/rssp-dev-cdn/test/rokt2.html) provided by Rakuten while mocking the same-domain nested iframe presentation we observed in ad units served through RUNA.
