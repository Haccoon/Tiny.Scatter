# Tiny.Scatter
Scatter javascript warpper for webview

## inject to WKWebView
```swift
extension WKWebViewConfiguration {
    static func makeScatterEOSSupport(account: String, publicKey: String, in messageHandler: WKScriptMessageHandler, with config: WKWebViewConfiguration) -> Void {
        var js = ""
        
        if let filepath = Bundle.main.path(forResource: "tiny_scatter", ofType: "js") {
            do {
                js += try String(contentsOfFile: filepath)
            } catch { }
        }
        
        js +=
        """
        
        // value as string with this format '{"signatures":["SIG_K1_..."]}'
        function onSignEOSSuccessful(id, value) {
            BrigeAPI.sendResponse(id, JSON.parse(value))
        }
        
        // error as string
        function onSignEOSError(id, error) {
            BrigeAPI.sendError(id, {"type": "signature_rejected", "message": error, "code": 402, "isError": true})
        }

        TinyIdentitys.initEOS("\(account)", "\(publicKey)");
        
        const scatter = new TinyScatter();
        scatter.loadPlugin(new TinyEOS());
        
        window.scatter = scatter;
        
        document.dispatchEvent(new CustomEvent('scatterLoaded'));

        """
        let userScript = WKUserScript(source: js, injectionTime: .atDocumentStart, forMainFrameOnly: false)
        config.userContentController.add(messageHandler, name: XMethod.signEOS.rawValue)
        config.userContentController.addUserScript(userScript)
    }
}
```
