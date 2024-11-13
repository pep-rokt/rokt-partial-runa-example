# ROKT x RUNA Integration Example

This repo is an example Rokt integration that is served via Rakuten's RUNA. To demo, simply run `live-server` at the project root. 

An important note - based on conversations with the RUNA team, this example assumes that all RUNA iframes share the same domain & sub-domain with the top-level application. 

Breaking down the example...
- The Rokt integration is executed via a single script delivered via a RUNA tag:

  ```html
  <script type="module">
    // access top-level document (assuming top-level app and iframes are same-domain)
    const top = window.top.document;

    // load Rokt wSDK & create Rokt launcher in top-level document
    const roktLauncherScriptPromise = new Promise((resolve, reject) => {
      const target = top.head || top.body;
      const script = top.createElement("script");
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

    // Use top-level Rokt launcher to fetch offers 
    // Rokt Layout will render at the top-level where the SDK where the rokt-contorller-frame exists
    await roktLauncherScriptPromise;
    const launcher = await window.top.Rokt.createLauncher({
      accountId: "2921597674431243178",
      sandbox: true, // remove when going live
    });

    await launcher.selectPlacements({
      // attributes are hardcoded as example; please map to correct values available through RUNA
      attributes: {
        email: "customer@gmail.com",
        firstname: "John",
        lastname: "Doe",
        confirmationref: "12345",
      }
    });
  </script>
  ```
- On execution, the script loads the Rokt WebSDK and creates the Rokt launcher in the top-level document. 
- Rokt's WebSDK also scaffolds a few iframes necessary for the integration into the top-level document, including the `rokt-controller-frame` and `Rokt placement` iframes. The `Rokt placement` iframe is responsible for rendering Rokt offers 
- Once the Rokt launcher is available, the script calls launcher.selectPlacements to fetch eligible Rokt offers to be rendered at the top-level. 
- Since the Rokt iframes exist at the top-level, the Rokt Overlay will also get rendered at the top-level of the application (outside of any RUNA iframe)
- Note that the RUNA iframes in this example were hardcoded (and not injected via the `rdntag.display()` call). This is to override the [sandbox Rokt integration](https://storage.googleapis.com/rssp-dev-cdn/test/rokt2.html) provided by Rakuten while mocking the same-domain nested iframe presentation we observed in ad units served through RUNA.
