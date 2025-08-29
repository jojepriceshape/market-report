
Apollo.io
<html lang="en"><head><script>(function(firebaseConfig, initialAuthToken, appId) {
        window.__firebase_config = firebaseConfig;
        window.__initial_auth_token = initialAuthToken;
        window.__app_id = appId;
            })("\n{\n  \"apiKey\": \"AIzaSyCqyCcs2R2e7AegGjvFAwG98wlamtbHvZY\",\n  \"authDomain\": \"bard-frontend.firebaseapp.com\",\n  \"projectId\": \"bard-frontend\",\n  \"storageBucket\": \"bard-frontend.firebasestorage.app\",\n  \"messagingSenderId\": \"175205271074\",\n  \"appId\": \"1:175205271074:web:2b7bd4d34d33bf38e6ec7b\"\n}\n","eyJhbGciOiJSUzI1NiIsImtpZCI6ImY3NWQ2NDhiMmFkZjUwOWU5MTEzMWVlYjQ2MzRlYTBhNGUxOTg3ZDUiLCJ0eXAiOiJKV1QifQ.eyJzdWIiOiJmaXJlYmFzZS1hZG1pbnNkay1mYnN2Y0BiYXJkLWZyb250ZW5kLmlhbS5nc2VydmljZWFjY291bnQuY29tIiwiYXVkIjoiaHR0cHM6Ly9pZGVudGl0eXRvb2xraXQuZ29vZ2xlYXBpcy5jb20vZ29vZ2xlLmlkZW50aXR5LmlkZW50aXR5dG9vbGtpdC52MS5JZGVudGl0eVRvb2xraXQiLCJ1aWQiOiIwOTg0MDY2MTU3Mjg5MjgxNDAyNyIsImlzcyI6ImZpcmViYXNlLWFkbWluc2RrLWZic3ZjQGJhcmQtZnJvbnRlbmQuaWFtLmdzZXJ2aWNlYWNjb3VudC5jb20iLCJjbGFpbXMiOnsiYXBwSWQiOiJjXzk1NDBjN2FkMmQ5MjdhMzlfbWFya2V0LXJlcG9ydC01NjUifSwiZXhwIjoxNzU1ODUzNDU0LCJpYXQiOjE3NTU4NDk4NTQsImFsZyI6IlJTMjU2In0.lA3UhVmzt1rPZvG--SuKJAWgvugunwTbISRc_4GqohENrlqmb1a4DpcVsXX7AXmdbz7rJdWsK-CVUfPDx408pJFHbcvxVXvGiWfdCLd3yoIelFmjCT5TecI9R-r52Ex1kcWr8TlmFOgsYv1jZ25yXSJp8L_lfVWmzHp5DCRm4mgJE3Gn-1hI6b5uGIGeMKCCncemsHoUCwhcBRcdTy6pI3XcfM3Lje--FQQAgX-XTd4s-7-LihWPNohdZChfrR1qw8A2DD-tdQ3rXAktKBkuvWC08dG9kLZfMDWlV7SxtkWHkav77Tiz3nwFy4MxhabvZU1lqTEDwzGAOi_zC9cIgw","c_9540c7ad2d927a39_market-report-565")</script><script>(function() {
  // Ensure this script is executed only once
  if (window.firebaseAuthBridgeScriptLoaded) {
    return;
  }
  window.firebaseAuthBridgeScriptLoaded = true;

  let nextTokenPromiseId = 0;

  // Stores { resolve, reject } for ongoing token requests
  const pendingTokenPromises = {};

  // Listen for messages from the Host Application
  window.addEventListener('message', function(event) {

    const messageData = event.data;

  if (messageData && messageData.type === 'RESOLVE_NEW_FIREBASE_TOKEN') {
      const { success, token, error, promiseId } = messageData ?? {};
      if (pendingTokenPromises[promiseId]) {
        if (success) {
          pendingTokenPromises[promiseId].resolve(token);
        } else {
          pendingTokenPromises[promiseId].reject(new Error(error || 'Token refresh failed from host.'));
        }
        delete pendingTokenPromises[promiseId];
      }
    }
  });

  // Expose a function for the Generated App to request a new Firebase token
  window.requestNewFirebaseToken = function() {
    const currentPromiseId = nextTokenPromiseId++;
    const promise = new Promise((resolve, reject) => {
      pendingTokenPromises[currentPromiseId] = { resolve, reject };
    });
    if (window.parent && window.parent !== window) {
      window.parent.postMessage({
        type: 'REQUEST_NEW_FIREBASE_TOKEN',
        promiseId: currentPromiseId
      }, '*');
    } else {
      pendingTokenPromises[currentPromiseId].reject(new Error('No parent window to request token from.'));
      delete pendingTokenPromises[currentPromiseId];
    }
    return promise;
  };
})();</script><script>
let realOriginalGetUserMedia = null;
if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
  realOriginalGetUserMedia = navigator.mediaDevices.getUserMedia.bind(navigator.mediaDevices);
}

(function() {
  if (navigator.mediaDevices && navigator.mediaDevices.__proto__) {
    try {
      Object.defineProperty(navigator.mediaDevices.__proto__, 'getUserMedia', {
        get: function() {
          return undefined; // Or throw an error
        },
        configurable: false
      });
    } catch (error) {
      console.error("Error defining prototype getter:", error);
    }
  }
})();

(function() {
  const pendingMediaResolvers = {};
  let nextMediaPromiseId = 0;

  function requestMediaPermissions(constraints) {
    const mediaPromiseId = nextMediaPromiseId++;
    const promise = new Promise((resolve, reject) => {
      pendingMediaResolvers[mediaPromiseId] = (granted) => {
        delete pendingMediaResolvers[mediaPromiseId];
        resolve(granted);
      };
    });

    window.parent.postMessage({
      type: 'requestMediaPermission',
      constraints: constraints,
      promiseId: mediaPromiseId,
    }, '*');

    return promise;
  }

  let originalGetUserMedia = realOriginalGetUserMedia;

  function interceptGetUserMedia() {
    if (navigator.mediaDevices) {
      Object.defineProperty(navigator.mediaDevices, 'getUserMedia', {
        value: function(constraints) {
          return requestMediaPermissions(constraints).then((granted) => {
            if (granted) {
              if (originalGetUserMedia) {
                return originalGetUserMedia(constraints);
              } else {
                throw new Error("Original getUserMedia not available.");
              }
            } else {
              throw new DOMException('Permission denied', 'NotAllowedError');
            }
          });
        },
        writable: false,
        configurable: false
      });
    }
  }

  interceptGetUserMedia();

  const observer = new MutationObserver(function(mutationsList, observer) {
    for (const mutation of mutationsList) {
      if (mutation.type === 'reconfigured' && mutation.name === 'getUserMedia' && mutation.object === navigator.mediaDevices) {
        interceptGetUserMedia();
      } else if (mutation.type === 'attributes' && mutation.attributeName === 'getUserMedia' && mutation.target === navigator.mediaDevices) {
        interceptGetUserMedia();
      } else if (mutation.type === 'childList' && mutation.addedNodes) {
        mutation.addedNodes.forEach(node => {
          if (node === navigator.mediaDevices) {
            interceptGetUserMedia();
          }
        });
      }
    }
  });

  function interceptSpeechRecognition() {
    if (!window.SpeechRecognition && !window.webkitSpeechRecognition) {
      return;
    }

    const OriginalSpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;

    const SpeechRecognitionWrapper = function(...args) {
      const recognizer = new OriginalSpeechRecognition(...args);
      const originalStart = recognizer.start.bind(recognizer);

      recognizer.start = function() {
        requestMediaPermissions({ audio: true }).then(granted => {
          if (granted) {
            originalStart();
          } else {
            const errorEvent = new SpeechRecognitionErrorEvent('error');
            errorEvent.error = 'not-allowed'; // This is the standard error for permission denial.
            recognizer.dispatchEvent(errorEvent);
          }
        });
      };

      return recognizer;
    };

    SpeechRecognitionWrapper.prototype = OriginalSpeechRecognition.prototype;
    SpeechRecognitionWrapper.prototype.constructor = SpeechRecognitionWrapper;

    if (window.SpeechRecognition) {
      window.SpeechRecognition = SpeechRecognitionWrapper;
    }
    if (window.webkitSpeechRecognition) {
      window.webkitSpeechRecognition = SpeechRecognitionWrapper;
    }
  }

  interceptSpeechRecognition();

  window.addEventListener('message', function(event) {
    if (event.data) {
      if (event.data.type === 'resolveMediaPermission') {
        const { promiseId, granted } = event.data;
        if (pendingMediaResolvers[promiseId]) {
          pendingMediaResolvers[promiseId](granted);
        }
      }
    }
  });

})();</script><script>((function(modelInformation) {
  const originalFetch = window.fetch;
  // TODO: b/421908508 - Move these out of the script and match all generative AI model calls.
  let googleLlmBaseApiUrls = [
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.textModelName + ':streamGenerateContent',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.textModelName + ':generateContent',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.imageModelName + ':predict',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.imageModelName + ':predictLongRunning',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.imageEditModelName + ':generateContent',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.videoModelName + ':predict',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.videoModelName + ':predictLongRunning',
    'https://generativelanguage.googleapis.com/v1beta/models/' + modelInformation.ttsModelName + ':generateContent',
  ];
  modelInformation.deprecatedTextModelNames.forEach((modelName) => {
    googleLlmBaseApiUrls.push(
      'https://generativelanguage.googleapis.com/v1beta/models/' + modelName + ':streamGenerateContent',
      'https://generativelanguage.googleapis.com/v1beta/models/' + modelName + ':generateContent',
    );
  });

  const pendingFetchResolvers = {};
  let nextPromiseId = 0;

  function handleStringInput(input, optionsArgument) {
    const actualUrl = input;
    const fetchCallArgs = [actualUrl, optionsArgument];
    const effectiveOptions = optionsArgument || {};
    const bodyForApiKeyCheck = effectiveOptions.body;
    const bodyForPostMessage = effectiveOptions.body;
    return { actualUrl, fetchCallArgs, effectiveOptions, bodyForApiKeyCheck, bodyForPostMessage };
  }

  function handleRequestInput(input, optionsArgument) {
    const actualUrl = input.url;
    const fetchCallArgs = [input, optionsArgument];
    const effectiveOptions = { method: input.method, headers: new Headers(input.headers) };
    let bodyForApiKeyCheck;
    let bodyForPostMessage;

    if (optionsArgument) {
      if (optionsArgument.method) effectiveOptions.method = optionsArgument.method;
      if (optionsArgument.headers) effectiveOptions.headers = new Headers(optionsArgument.headers);
      if ('body' in optionsArgument) {
        bodyForApiKeyCheck = optionsArgument.body;
        bodyForPostMessage = optionsArgument.body;
      } else {
        bodyForApiKeyCheck = undefined;
        bodyForPostMessage = input.body;
      }
    } else {
      bodyForApiKeyCheck = undefined;
      bodyForPostMessage = input.body;
    }
    return { actualUrl, fetchCallArgs, effectiveOptions, bodyForApiKeyCheck, bodyForPostMessage };
  }

  window.fetch = function(input, optionsArgument) {
    let actualUrl;
    let fetchCallArgs;
    let effectiveOptions = {};
    let bodyForApiKeyCheck;
    let bodyForPostMessage;

    if (typeof input === 'string') {
      ({actualUrl, fetchCallArgs, effectiveOptions, bodyForApiKeyCheck, bodyForPostMessage} = handleStringInput(input, optionsArgument));
    } else if (input instanceof Request) {
      ({actualUrl, fetchCallArgs, effectiveOptions, bodyForApiKeyCheck, bodyForPostMessage} = handleRequestInput(input, optionsArgument));
    } else {
      return originalFetch.apply(window, [input, optionsArgument]);
    }

    effectiveOptions.method = effectiveOptions.method || 'GET';
    if (!effectiveOptions.headers) {
      effectiveOptions.headers = new Headers();
    }


    if (typeof actualUrl === 'string' && googleLlmBaseApiUrls.some((url) => actualUrl.startsWith(url))) {
      let apiKeyIsNull = true;

      const regex = new RegExp("models/([^:]+)");
      const modelNameMatch = actualUrl.match(regex);
      const modelName = modelNameMatch ? modelNameMatch[1] : 'unspecified';


      try {
        const urlObject = new URL(actualUrl);  // Use URL object for robust parsing
        const apiKeyParam = urlObject.searchParams.get('key');
        if (apiKeyParam) {
          apiKeyIsNull = false;
        }
      } catch (e) {
        // Continue checks even if URL parsing fails
      }

      if (apiKeyIsNull && effectiveOptions.headers) {
        const h = new Headers(effectiveOptions.headers);
        const apiKeyHeaderValue = h.get('X-API-Key') || h.get('x-api-key');
        if (apiKeyHeaderValue) {
          apiKeyIsNull = false;
          return originalFetch.apply(window, fetchCallArgs);
        }
      }

      if (apiKeyIsNull && effectiveOptions.method && ['POST', 'PUT', 'PATCH'].includes(effectiveOptions.method.toUpperCase()) && typeof bodyForApiKeyCheck === 'string') {
        try {
          const bodyData = JSON.parse(bodyForApiKeyCheck);
          if (bodyData && bodyData.apiKey) {
            apiKeyIsNull = false;
            return originalFetch.apply(window, fetchCallArgs);
          }
        } catch (e) {
          // Ignore JSON parsing errors
        }
      }

      if(apiKeyIsNull) {
        const promiseId = nextPromiseId++;
        const promise = new Promise((resolve) => {
          pendingFetchResolvers[promiseId] = (resolvedResponse) => {
            delete pendingFetchResolvers[promiseId];
            resolve(resolvedResponse);
          };
        });

        let serializedBodyForPostMessage;
        if (typeof bodyForPostMessage === 'string' || bodyForPostMessage == null) {
            serializedBodyForPostMessage = bodyForPostMessage;
        } else if (bodyForPostMessage instanceof ReadableStream) {
            serializedBodyForPostMessage = null;
        } else {
            try {
                serializedBodyForPostMessage = JSON.stringify(bodyForPostMessage);
            } catch (e) {
                serializedBodyForPostMessage = null;
            }
        }

        const messageOptions = {
            method: effectiveOptions.method,
            headers: Object.fromEntries(new Headers(effectiveOptions.headers).entries()),
            body: serializedBodyForPostMessage
        };

        window.parent.postMessage({
          type: 'requestFetch',
          url: actualUrl,
          modelName: modelName,
          options: messageOptions,
          promiseId: promiseId,
        }, '*');

        return promise;
      }
      return originalFetch.apply(window, fetchCallArgs);
    }
    return originalFetch.apply(window, fetchCallArgs);
  };

  window.addEventListener('message', function(event) {
    if (event.data && event.data.type === 'resolveFetch') {
      const { promiseId, response } = event.data;
      if (pendingFetchResolvers[promiseId]) {
        try {
          const reconstructedResponse = new Response(response.body, {
            status: response.status,
            statusText: response.statusText,
            headers: new Headers(response.headers),
          });
          pendingFetchResolvers[promiseId](reconstructedResponse);
        } catch (error) {
          pendingFetchResolvers[promiseId](new Response(null, { status: 500, statusText: "Interceptor Response Reconstruction Error" }));
        }
      }
    }
  });

}))({"textModelName":"gemini-2.5-flash-preview-05-20","imageModelName":"imagen-3.0-generate-002","imageEditModelName":"gemini-2.0-flash-preview-image-generation","videoModelName":"veo-2.0-generate-001","ttsModelName":"gemini-2.5-flash-preview-tts","deprecatedTextModelNames":["gemini-2.0-flash","gemini-2.5-flash-preview-04-17"]})</script><script>(function() {
  const originalConsoleLog = console.log;
  const originalConsoleError = console.error;

    /**
   * Normalizes an error event or a promise rejection reason into a structured error object.
   * @param {*} errorEventOrReason The error object or reason.
   * @return {object} Structured error data { message, name, stack }.
   */
  function getErrorObject(errorEventOrReason) {
    if (errorEventOrReason instanceof Error) {
      return {
        message: errorEventOrReason.message,
        name: errorEventOrReason.name,
        stack: errorEventOrReason.stack,
      };
    }
    // Fallback for non-Error objects.
    try {
      return {
        message: JSON.stringify(errorEventOrReason),
        name: 'UnknownErrorType',
        stack: null,
      };
    } catch (e) {
      return {
        message: String(errorEventOrReason),
        name: 'UnknownErrorTypeNonStringifiable',
        stack: null,
      };
    }
  }

  /**
   * Converts an array of arguments (from log/error) into a single string.
   * Handles Error objects specially to include their message and stack.
   * @param {Array<*>} args - Arguments passed to console methods.
   * @return {string} A string representation of the arguments.
   */
  function stringifyArgs(args) {
    return args
      .map((arg) => {
        if (arg instanceof Error) {
          const {message, stack} = arg;
          return `Error: ${message}${stack ? ('\nStack: ' + stack) : ''}`;
        }
        if (typeof arg === 'object' && arg !== null) {
          try {
            return JSON.stringify(arg);
          } catch (error) {
            return '[Circular Object]';
          }
        } else {
          return String(arg);
        }
      })
      .join(' ');
  }

  console.log = function(...args) {
    const logString = stringifyArgs(args);
    window.parent.postMessage({ type: 'log', message: logString }, '*');
    originalConsoleLog.apply(console, args);
  };

  console.error = function(...args) {
    let errorData;
    if (args.length > 0 && args[0] instanceof Error) {
      const err = args[0];
      // If the first arg is an Error, capture its details.
      errorData = {
        type: 'error',
        source: 'CONSOLE_ERROR',
        ...getErrorObject(err),
        rawArgsString: stringifyArgs(args.slice(1)),
        timestamp: new Date().toISOString(),
      };
    } else {
      // If not an Error object, treat all args as a general error message.
      errorData = {
        type: 'error',
        source: 'CONSOLE_ERROR',
        message: stringifyArgs(args),
        name: 'ConsoleLoggedError',
        stack: null,
        timestamp: new Date().toISOString(),
      };
    }
    window.parent.postMessage(errorData, '*');
    originalConsoleError.apply(console, args);
  };

  // Listen for global unhandled synchronous errors.
  window.addEventListener('error', function(event) {
    const errorDetails = event.error ? getErrorObject(event.error) : {
      message: event.message,
      name: 'GlobalError',
      stack: null,
      filename: event.filename,
      lineno: event.lineno,
      colno: event.colno,
    };

    window.parent.postMessage({
      type: 'error',
      source: 'global',
      ...errorDetails,
      message: errorDetails.message || event.message,
      timestamp: new Date().toISOString(),
    }, '*');
  });

  // Listen for unhandled promise rejections (asynchronous errors).
  window.addEventListener('unhandledrejection', function(event) {
    const errorDetails = getErrorObject(event.reason);

    window.parent.postMessage({
      type: 'error',
      source: 'unhandledrejection',
      ...errorDetails,
      message: errorDetails.message || 'Unhandled Promise Rejection',
      timestamp: new Date().toISOString(),
    }, '*');
  });

})();</script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Market Analysis Report</title>
    <!-- Use Inter font from Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&amp;display=swap" rel="stylesheet">
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* Custom scrollbar styling */
        ::-webkit-scrollbar {
            width: 8px;
        }

        ::-webkit-scrollbar-track {
            background: #1f2937;
        }

        ::-webkit-scrollbar-thumb {
            background: #4b5563;
            border-radius: 4px;
        }

        ::-webkit-scrollbar-thumb:hover {
            background: #6b7280;
        }

        body {
            font-family: 'Inter', sans-serif;
            -webkit-font-smoothing: antialiased;
            -moz-osx-font-smoothing: grayscale;
            /* New styling to allow for normal scrolling on a downloaded file */
            background-color: #111827;
            color: #d1d5db;
            padding: 2rem;
        }
    </style>
<style>*, ::before, ::after{--tw-border-spacing-x:0;--tw-border-spacing-y:0;--tw-translate-x:0;--tw-translate-y:0;--tw-rotate:0;--tw-skew-x:0;--tw-skew-y:0;--tw-scale-x:1;--tw-scale-y:1;--tw-pan-x: ;--tw-pan-y: ;--tw-pinch-zoom: ;--tw-scroll-snap-strictness:proximity;--tw-gradient-from-position: ;--tw-gradient-via-position: ;--tw-gradient-to-position: ;--tw-ordinal: ;--tw-slashed-zero: ;--tw-numeric-figure: ;--tw-numeric-spacing: ;--tw-numeric-fraction: ;--tw-ring-inset: ;--tw-ring-offset-width:0px;--tw-ring-offset-color:#fff;--tw-ring-color:rgb(59 130 246 / 0.5);--tw-ring-offset-shadow:0 0 #0000;--tw-ring-shadow:0 0 #0000;--tw-shadow:0 0 #0000;--tw-shadow-colored:0 0 #0000;--tw-blur: ;--tw-brightness: ;--tw-contrast: ;--tw-grayscale: ;--tw-hue-rotate: ;--tw-invert: ;--tw-saturate: ;--tw-sepia: ;--tw-drop-shadow: ;--tw-backdrop-blur: ;--tw-backdrop-brightness: ;--tw-backdrop-contrast: ;--tw-backdrop-grayscale: ;--tw-backdrop-hue-rotate: ;--tw-backdrop-invert: ;--tw-backdrop-opacity: ;--tw-backdrop-saturate: ;--tw-backdrop-sepia: ;--tw-contain-size: ;--tw-contain-layout: ;--tw-contain-paint: ;--tw-contain-style: }::backdrop{--tw-border-spacing-x:0;--tw-border-spacing-y:0;--tw-translate-x:0;--tw-translate-y:0;--tw-rotate:0;--tw-skew-x:0;--tw-skew-y:0;--tw-scale-x:1;--tw-scale-y:1;--tw-pan-x: ;--tw-pan-y: ;--tw-pinch-zoom: ;--tw-scroll-snap-strictness:proximity;--tw-gradient-from-position: ;--tw-gradient-via-position: ;--tw-gradient-to-position: ;--tw-ordinal: ;--tw-slashed-zero: ;--tw-numeric-figure: ;--tw-numeric-spacing: ;--tw-numeric-fraction: ;--tw-ring-inset: ;--tw-ring-offset-width:0px;--tw-ring-offset-color:#fff;--tw-ring-color:rgb(59 130 246 / 0.5);--tw-ring-offset-shadow:0 0 #0000;--tw-ring-shadow:0 0 #0000;--tw-shadow:0 0 #0000;--tw-shadow-colored:0 0 #0000;--tw-blur: ;--tw-brightness: ;--tw-contrast: ;--tw-grayscale: ;--tw-hue-rotate: ;--tw-invert: ;--tw-saturate: ;--tw-sepia: ;--tw-drop-shadow: ;--tw-backdrop-blur: ;--tw-backdrop-brightness: ;--tw-backdrop-contrast: ;--tw-backdrop-grayscale: ;--tw-backdrop-hue-rotate: ;--tw-backdrop-invert: ;--tw-backdrop-opacity: ;--tw-backdrop-saturate: ;--tw-backdrop-sepia: ;--tw-contain-size: ;--tw-contain-layout: ;--tw-contain-paint: ;--tw-contain-style: }/* ! tailwindcss v3.4.17 | MIT License | https://tailwindcss.com */*,::after,::before{box-sizing:border-box;border-width:0;border-style:solid;border-color:#e5e7eb}::after,::before{--tw-content:''}:host,html{line-height:1.5;-webkit-text-size-adjust:100%;-moz-tab-size:4;tab-size:4;font-family:ui-sans-serif, system-ui, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";font-feature-settings:normal;font-variation-settings:normal;-webkit-tap-highlight-color:transparent}body{margin:0;line-height:inherit}hr{height:0;color:inherit;border-top-width:1px}abbr:where([title]){-webkit-text-decoration:underline dotted;text-decoration:underline dotted}h1,h2,h3,h4,h5,h6{font-size:inherit;font-weight:inherit}a{color:inherit;text-decoration:inherit}b,strong{font-weight:bolder}code,kbd,pre,samp{font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;font-feature-settings:normal;font-variation-settings:normal;font-size:1em}small{font-size:80%}sub,sup{font-size:75%;line-height:0;position:relative;vertical-align:baseline}sub{bottom:-.25em}sup{top:-.5em}table{text-indent:0;border-color:inherit;border-collapse:collapse}button,input,optgroup,select,textarea{font-family:inherit;font-feature-settings:inherit;font-variation-settings:inherit;font-size:100%;font-weight:inherit;line-height:inherit;letter-spacing:inherit;color:inherit;margin:0;padding:0}button,select{text-transform:none}button,input:where([type=button]),input:where([type=reset]),input:where([type=submit]){-webkit-appearance:button;background-color:transparent;background-image:none}:-moz-focusring{outline:auto}:-moz-ui-invalid{box-shadow:none}progress{vertical-align:baseline}::-webkit-inner-spin-button,::-webkit-outer-spin-button{height:auto}[type=search]{-webkit-appearance:textfield;outline-offset:-2px}::-webkit-search-decoration{-webkit-appearance:none}::-webkit-file-upload-button{-webkit-appearance:button;font:inherit}summary{display:list-item}blockquote,dd,dl,figure,h1,h2,h3,h4,h5,h6,hr,p,pre{margin:0}fieldset{margin:0;padding:0}legend{padding:0}menu,ol,ul{list-style:none;margin:0;padding:0}dialog{padding:0}textarea{resize:vertical}input::placeholder,textarea::placeholder{opacity:1;color:#9ca3af}[role=button],button{cursor:pointer}:disabled{cursor:default}audio,canvas,embed,iframe,img,object,svg,video{display:block;vertical-align:middle}img,video{max-width:100%;height:auto}[hidden]:where(:not([hidden=until-found])){display:none}.fixed{position:fixed}.bottom-6{bottom:1.5rem}.right-6{right:1.5rem}.mx-auto{margin-left:auto;margin-right:auto}.mb-3{margin-bottom:0.75rem}.mb-4{margin-bottom:1rem}.mr-2{margin-right:0.5rem}.mt-1{margin-top:0.25rem}.mt-12{margin-top:3rem}.mt-2{margin-top:0.5rem}.mt-4{margin-top:1rem}.inline-block{display:inline-block}.flex{display:flex}.grid{display:grid}.h-1\.5{height:0.375rem}.h-6{height:1.5rem}.w-6{width:1.5rem}.w-full{width:100%}.max-w-4xl{max-width:56rem}.transform{transform:translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))}.grid-cols-1{grid-template-columns:repeat(1, minmax(0, 1fr))}.items-center{align-items:center}.justify-between{justify-content:space-between}.gap-6{gap:1.5rem}.space-y-12 > :not([hidden]) ~ :not([hidden]){--tw-space-y-reverse:0;margin-top:calc(3rem * calc(1 - var(--tw-space-y-reverse)));margin-bottom:calc(3rem * var(--tw-space-y-reverse))}.space-y-2 > :not([hidden]) ~ :not([hidden]){--tw-space-y-reverse:0;margin-top:calc(0.5rem * calc(1 - var(--tw-space-y-reverse)));margin-bottom:calc(0.5rem * var(--tw-space-y-reverse))}.space-y-6 > :not([hidden]) ~ :not([hidden]){--tw-space-y-reverse:0;margin-top:calc(1.5rem * calc(1 - var(--tw-space-y-reverse)));margin-bottom:calc(1.5rem * var(--tw-space-y-reverse))}.rounded-2xl{border-radius:1rem}.rounded-full{border-radius:9999px}.rounded-lg{border-radius:0.5rem}.rounded-xl{border-radius:0.75rem}.border{border-width:1px}.border-t{border-top-width:1px}.border-gray-700{--tw-border-opacity:1;border-color:rgb(55 65 81 / var(--tw-border-opacity, 1))}.border-gray-700\/50{border-color:rgb(55 65 81 / 0.5)}.bg-blue-400{--tw-bg-opacity:1;background-color:rgb(96 165 250 / var(--tw-bg-opacity, 1))}.bg-blue-600{--tw-bg-opacity:1;background-color:rgb(37 99 235 / var(--tw-bg-opacity, 1))}.bg-gray-600{--tw-bg-opacity:1;background-color:rgb(75 85 99 / var(--tw-bg-opacity, 1))}.bg-gray-700{--tw-bg-opacity:1;background-color:rgb(55 65 81 / var(--tw-bg-opacity, 1))}.bg-gray-700\/50{background-color:rgb(55 65 81 / 0.5)}.bg-gray-800{--tw-bg-opacity:1;background-color:rgb(31 41 55 / var(--tw-bg-opacity, 1))}.bg-gray-800\/50{background-color:rgb(31 41 55 / 0.5)}.bg-gray-900{--tw-bg-opacity:1;background-color:rgb(17 24 39 / var(--tw-bg-opacity, 1))}.bg-green-600{--tw-bg-opacity:1;background-color:rgb(22 163 74 / var(--tw-bg-opacity, 1))}.bg-red-400{--tw-bg-opacity:1;background-color:rgb(248 113 113 / var(--tw-bg-opacity, 1))}.p-4{padding:1rem}.p-5{padding:1.25rem}.p-6{padding:1.5rem}.p-8{padding:2rem}.px-8{padding-left:2rem;padding-right:2rem}.py-3{padding-top:0.75rem;padding-bottom:0.75rem}.pt-12{padding-top:3rem}.pt-8{padding-top:2rem}.text-center{text-align:center}.text-2xl{font-size:1.5rem;line-height:2rem}.text-3xl{font-size:1.875rem;line-height:2.25rem}.text-lg{font-size:1.125rem;line-height:1.75rem}.text-sm{font-size:0.875rem;line-height:1.25rem}.text-xl{font-size:1.25rem;line-height:1.75rem}.text-xs{font-size:0.75rem;line-height:1rem}.font-bold{font-weight:700}.font-medium{font-weight:500}.font-semibold{font-weight:600}.tracking-tight{letter-spacing:-0.025em}.text-blue-100\/80{color:rgb(219 234 254 / 0.8)}.text-blue-400{--tw-text-opacity:1;color:rgb(96 165 250 / var(--tw-text-opacity, 1))}.text-gray-100{--tw-text-opacity:1;color:rgb(243 244 246 / var(--tw-text-opacity, 1))}.text-gray-200{--tw-text-opacity:1;color:rgb(229 231 235 / var(--tw-text-opacity, 1))}.text-gray-300{--tw-text-opacity:1;color:rgb(209 213 219 / var(--tw-text-opacity, 1))}.text-gray-400{--tw-text-opacity:1;color:rgb(156 163 175 / var(--tw-text-opacity, 1))}.text-gray-500{--tw-text-opacity:1;color:rgb(107 114 128 / var(--tw-text-opacity, 1))}.text-green-400{--tw-text-opacity:1;color:rgb(74 222 128 / var(--tw-text-opacity, 1))}.text-red-400{--tw-text-opacity:1;color:rgb(248 113 113 / var(--tw-text-opacity, 1))}.text-white{--tw-text-opacity:1;color:rgb(255 255 255 / var(--tw-text-opacity, 1))}.text-yellow-400{--tw-text-opacity:1;color:rgb(250 204 21 / var(--tw-text-opacity, 1))}.shadow-2xl{--tw-shadow:0 25px 50px -12px rgb(0 0 0 / 0.25);--tw-shadow-colored:0 25px 50px -12px var(--tw-shadow-color);box-shadow:var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)}.shadow-inner{--tw-shadow:inset 0 2px 4px 0 rgb(0 0 0 / 0.05);--tw-shadow-colored:inset 0 2px 4px 0 var(--tw-shadow-color);box-shadow:var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)}.shadow-lg{--tw-shadow:0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);--tw-shadow-colored:0 10px 15px -3px var(--tw-shadow-color), 0 4px 6px -4px var(--tw-shadow-color);box-shadow:var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)}.transition-all{transition-property:all;transition-timing-function:cubic-bezier(0.4, 0, 0.2, 1);transition-duration:150ms}.transition-transform{transition-property:transform;transition-timing-function:cubic-bezier(0.4, 0, 0.2, 1);transition-duration:150ms}.duration-200{transition-duration:200ms}.duration-300{transition-duration:300ms}.duration-500{transition-duration:500ms}.hover\:scale-105:hover{--tw-scale-x:1.05;--tw-scale-y:1.05;transform:translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))}.hover\:scale-110:hover{--tw-scale-x:1.1;--tw-scale-y:1.1;transform:translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))}.hover\:scale-\[1\.01\]:hover{--tw-scale-x:1.01;--tw-scale-y:1.01;transform:translate(var(--tw-translate-x), var(--tw-translate-y)) rotate(var(--tw-rotate)) skewX(var(--tw-skew-x)) skewY(var(--tw-skew-y)) scaleX(var(--tw-scale-x)) scaleY(var(--tw-scale-y))}.hover\:bg-gray-700:hover{--tw-bg-opacity:1;background-color:rgb(55 65 81 / var(--tw-bg-opacity, 1))}.hover\:bg-green-700:hover{--tw-bg-opacity:1;background-color:rgb(21 128 61 / var(--tw-bg-opacity, 1))}.hover\:shadow-lg:hover{--tw-shadow:0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);--tw-shadow-colored:0 10px 15px -3px var(--tw-shadow-color), 0 4px 6px -4px var(--tw-shadow-color);box-shadow:var(--tw-ring-offset-shadow, 0 0 #0000), var(--tw-ring-shadow, 0 0 #0000), var(--tw-shadow)}.focus\:outline-none:focus{outline:2px solid transparent;outline-offset:2px}.focus\:ring-2:focus{--tw-ring-offset-shadow:var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);--tw-ring-shadow:var(--tw-ring-inset) 0 0 0 calc(2px + var(--tw-ring-offset-width)) var(--tw-ring-color);box-shadow:var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000)}.focus\:ring-blue-500:focus{--tw-ring-opacity:1;--tw-ring-color:rgb(59 130 246 / var(--tw-ring-opacity, 1))}.focus\:ring-offset-2:focus{--tw-ring-offset-width:2px}.focus\:ring-offset-gray-900:focus{--tw-ring-offset-color:#111827}@media (min-width: 640px){.sm\:text-4xl{font-size:2.25rem;line-height:2.5rem}.sm\:text-base{font-size:1rem;line-height:1.5rem}}@media (min-width: 768px){.md\:grid-cols-2{grid-template-columns:repeat(2, minmax(0, 1fr))}.md\:p-8{padding:2rem}}</style></head>
<body class="bg-gray-900 text-gray-300 p-8">
    <div class="mx-auto w-full max-w-4xl bg-gray-800 rounded-2xl shadow-2xl transform transition-transform duration-500 hover:scale-[1.01]">
        <!-- Header Section -->
        <div class="bg-blue-600 p-6 md:p-8 text-white">
            <h1 class="text-3xl sm:text-4xl font-bold tracking-tight">Market Analysis Report</h1>
            <p class="mt-2 text-blue-100/80 text-sm sm:text-base">
                3 Easy Wins to Activate Sales
            </p>
        </div>

        <!-- Main Content -->
        <div class="p-6 md:p-8 space-y-12">
            <!-- Summary Section -->
            <div class="bg-gray-700/50 rounded-xl p-6 md:p-8 space-y-6">
                <h2 class="text-2xl font-semibold text-gray-100 flex items-center">
                    <span class="mr-2 text-blue-400">📊</span> Your Current Market Situation
                </h2>
                <p class="text-sm sm:text-base text-gray-400">
                    This report analyzes your current market position for products above 100 DKK and identifies immediate opportunities to boost sales performance.
                </p>
                
                <!-- Metrics Grid -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                    <!-- Overall Market Position Card -->
                    <div class="bg-gray-800/50 rounded-lg p-5 border border-gray-700/50 shadow-inner">
                        <h3 class="text-lg font-medium text-gray-200 mb-3">Overall Market Position</h3>
                        <ul class="space-y-2 text-gray-400 text-sm">
                            <li>📈 Total products analyzed: <span class="text-gray-100 font-medium">19,533</span> products priced above 100 DKK</li>
                            <li>💰 Average price: <span class="text-gray-100 font-medium">1,164 DKK</span></li>
                            <li>
                                <div class="flex justify-between items-center">
                                    <span class="text-gray-300 font-medium">Market Leadership:</span>
                                    <span class="text-blue-400 font-semibold">13.5%</span>
                                </div>
                                <div class="w-full bg-gray-600 rounded-full h-1.5 mt-1">
                                    <div class="bg-blue-400 h-1.5 rounded-full" style="width: 13.5%"></div>
                                </div>
                            </li>
                            <li>📍 Average market position: <span class="text-gray-100 font-medium">5.6</span></li>
                            <li>
                                <div class="flex justify-between items-center">
                                    <span class="text-gray-300 font-medium">Major Challenge:</span>
                                    <span class="text-red-400 font-semibold">55.8%</span>
                                </div>
                                <div class="w-full bg-gray-600 rounded-full h-1.5 mt-1">
                                    <div class="bg-red-400 h-1.5 rounded-full" style="width: 55.8%"></div>
                                </div>
                            </li>
                        </ul>
                    </div>
                    
                    <!-- Stock Issues Card -->
                    <div class="bg-gray-800/50 rounded-lg p-5 border border-gray-700/50 shadow-inner">
                        <h3 class="text-lg font-medium text-gray-200 mb-3">Stock Issues</h3>
                        <ul class="space-y-2 text-gray-400 text-sm">
                            <li>
                                <div class="flex justify-between items-center">
                                    <span class="text-gray-300 font-medium">Out of Stock:</span>
                                    <span class="text-red-400 font-semibold">19.4%</span>
                                </div>
                                <div class="w-full bg-gray-600 rounded-full h-1.5 mt-1">
                                    <div class="bg-red-400 h-1.5 rounded-full" style="width: 19.4%"></div>
                                </div>
                            </li>
                            <li>📦 Missing sales on <span class="text-gray-100 font-medium">1,168</span> competitively-priced products.</li>
                        </ul>
                    </div>
                </div>
            </div>

            <!-- 3 Wins Section -->
            <div class="space-y-6">
                <h2 class="text-2xl font-semibold text-gray-100 flex items-center">
                    <span class="mr-2 text-green-400">🚀</span> 3 Easy Wins to Activate Sales
                </h2>
                
                <!-- Win 1 -->
                <div class="bg-gray-700/50 rounded-xl p-6 transition-all duration-300 hover:bg-gray-700 hover:shadow-lg">
                    <h3 class="text-lg font-medium text-gray-100">1. Restock Your Competitive Winners</h3>
                    <p class="mt-2 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Opportunity:</span> You have competitively-priced products (2nd-3rd place) that are out of stock. These are your easiest wins since the pricing is already competitive.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Action:</span> Prioritize restocking <span class="text-blue-400 font-semibold">Fein</span>, <span class="text-blue-400 font-semibold">Wagner</span>, <span class="text-blue-400 font-semibold">Rothenberger</span>, <span class="text-blue-400 font-semibold">Hikoki</span>, and <span class="text-blue-400 font-semibold">Festool</span> products that are already 2nd-3rd cheapest in market.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Impact:</span> Immediate sales activation on proven competitive products.
                    </p>
                </div>
                
                <!-- Win 2 -->
                <div class="bg-gray-700/50 rounded-xl p-6 transition-all duration-300 hover:bg-gray-700 hover:shadow-lg">
                    <h3 class="text-lg font-medium text-gray-100">2. Price Optimize Your Overpriced In-Stock Items</h3>
                    <p class="mt-2 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Opportunity:</span> You have high-value products that are significantly overpriced (15th-25th position) but in stock. Small price reductions could move these into competitive positions.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Action:</span> Focus on <span class="text-blue-400 font-semibold">Nilfisk</span> products priced 79,190-37,918 DKK that are ranked 15th-20th. Even moving to 5th-8th position could significantly boost sales.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Impact:</span> Transform non-performing high-value inventory into sales drivers.
                    </p>
                </div>

                <!-- Win 3 -->
                <div class="bg-gray-700/50 rounded-xl p-6 transition-all duration-300 hover:bg-gray-700 hover:shadow-lg">
                    <h3 class="text-lg font-medium text-gray-100">3. Leverage Your Near-Competitive Products</h3>
                    <p class="mt-2 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Opportunity:</span> You have 5,069 products in 2nd-3rd position that are already well-positioned. These need minimal effort to convert.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Action:</span> Consider small price adjustments on <span class="text-blue-400 font-semibold">Nilfisk</span>, <span class="text-blue-400 font-semibold">Festool</span>, <span class="text-blue-400 font-semibold">Proxxon</span>, <span class="text-blue-400 font-semibold">Karcher</span>, <span class="text-blue-400 font-semibold">Graco</span>, and <span class="text-blue-400 font-semibold">Raimondi</span> products to move them to #1 position, or enhance their visibility through better merchandising.
                    </p>
                    <p class="mt-1 text-sm text-gray-400">
                        <span class="font-bold text-gray-300">Impact:</span> Convert near-wins into market leaders with minimal investment.
                    </p>
                </div>
            </div>

            <!-- Expected Impact & Next Steps -->
            <div class="space-y-6">
                <h2 class="text-2xl font-semibold text-gray-100 flex items-center">
                    <span class="mr-2 text-yellow-400">🎯</span> Expected Impact &amp; Next Steps
                </h2>
                
                <div class="bg-gray-700/50 rounded-xl p-6">
                    <p class="font-bold text-lg text-gray-200">Quick Impact Estimate:</p>
                    <p class="mt-2 text-sm text-gray-400">
                        Focusing on these three areas could potentially activate sales on over <span class="text-green-400 font-bold">6,000+ products</span> with relatively minimal effort compared to major pricing overhauls.
                    </p>
                </div>
                
                <div class="bg-gray-700/50 rounded-xl p-6">
                    <p class="font-bold text-lg text-gray-200">Recommended Next Steps:</p>
                    <ul class="mt-2 text-sm space-y-2 text-gray-400">
                        <li>🗓️ <span class="font-semibold text-gray-300">Week 1:</span> Identify and restock top 20 competitive out-of-stock items.</li>
                        <li>🗓️ <span class="font-semibold text-gray-300">Week 2:</span> Analyze competitor pricing for overpriced Nilfisk products and adjust strategically.</li>
                        <li>🗓️ <span class="font-semibold text-gray-300">Week 3:</span> Review 2nd-3rd position products for quick optimization opportunities.</li>
                        <li>🗓️ <span class="font-semibold text-gray-300">Ongoing:</span> Monitor stock levels of competitive products to prevent future stockouts.</li>
                    </ul>
                </div>
            </div>

            <!-- New Call to Action for Meeting -->
            <div class="text-center pt-8 border-t border-gray-700 mt-12 pt-12">
                <h3 class="text-xl font-semibold text-gray-100 mb-4">Ready to discuss these insights?</h3>
                <a href="https://meetings-eu1.hubspot.com/jonas-jensen" class="inline-block bg-green-600 hover:bg-green-700 text-white font-medium py-3 px-8 rounded-full shadow-lg transition-all duration-300 transform hover:scale-105" target="_blank">
                    Book a Meeting
                </a>
                <p class="mt-4 text-xs text-gray-500">
                    Click the button above to schedule a call with Jonas.
                </p>
            </div>
        </div>
    </div>
    
    <!-- Download Report Button -->
    <button onclick="downloadReport()" class="fixed bottom-6 right-6 p-4 bg-gray-700 text-white rounded-full shadow-lg transition-transform duration-200 hover:scale-110 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 focus:ring-offset-gray-900">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path>
        </svg>
    </button>

    <!-- JavaScript for "download" functionality -->
    <script>
        function downloadReport() {
            // Get the HTML content of the entire document
            const htmlContent = document.documentElement.outerHTML;
            const blob = new Blob([htmlContent], { type: 'text/html' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'Market_Analysis_Report.html';
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
        }
    </script>


</body></html>
Viser Market_Analysis_Report (4) (2).html.
