## Adapting to the Head Unit Language

Adapting to the head unit language requires you to observe the register app interface response and use the `SDLChangeRegistration` RPC to update your app's name into the new language if you support it.

### When to support languages
Matches == The app's language and the head unit language are the same
Supported == The app supports the language returned by the head unit

| App Language | Head Unit Language  | What to do |
| ------------ | ------------------- | ---------- |
| Matches       | Supported   | Nothing, you already match |
| Matches       | Unsupported | This is impossible |
| Doesn't Match | Supported   | Switch the app to the head unit language |
| Doesn't Match | Unsupported | Nothing, continue using the app language |

So in that case where your app language doesn't match the head unit's language and you support the head unit's language, you will want to change your registration.

### Determining the Head Unit Language
After starting the `SDLManager`, the `SDLManager.registerResponse` property will exist. Check `SDLManager.registerResponse.language` to determine the current language.

### Changing your registration
You will use `SDLChangeRegistration` RPC to update your language.

#### Objective-C
```objc
SDLChangeRegistration *change = [[SDLChangeRegistration alloc] initWithLanguage:<#Matching language#> hmiDisplayLanguage:<#Matching language#> appName:@"<#App name for new language#>" ttsName:@[<#App TTS name for language#>] ngnMediaScreenAppName:nil vrSynonyms:nil];

[self.sdlManager.sendRequest:change withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
	if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
		// The change registration failed
		return;
	}
}
```

#### Swift
```swift
let change = SDLChangeRegistration(language:<#Matching language#>, hmiDisplayLanguage:<#Matching language#>, appName:"<#App name for new language#>" ttsName:[<#App TTS name for language#>], ngnMediaScreenAppName:nil, vrSynonyms:nil)

self.sdlManager.send(request: change) { (request, response, error) in
	if response?.resultCode != .success {
		// The change registration failed
		return
	}
}
```