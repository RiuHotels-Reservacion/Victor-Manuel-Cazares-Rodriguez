/* LuceCEM T2C v0.6.0 2021-12-28 */
if (!window.t2c) {
     if (!config_t2c_tag) config_t2c_tag = {
        // URL of the service
        "endpoint" : "https://track2call-riu.ew.r.appspot.com/get_available_phone",
        // URL of the endpoint to update the phonebooking
        "updateBookingEndpoint" : "https://track2call-riu.ew.r.appspot.com/update_availability",
        // Name of the cookie that is sent to the server to track the client activity
        "trackedCookie" : "_ga",
        // Selector ot the DOM element that triggers the request for a callcenter number (string or array of strings)
        "triggerSelectors" : ".trigger_t2c-phones",
        // Selector of the element that holds the phone number to update (string or array of strings)
        "targetSelectors" : ".target_t2c-phones",
        // Selector of the element that holds the phone number to update inside the href for mobile
        "targetHrefSelectors" : ".target_href_t2c-phones",
        // If not enabled, the retrieved phone number will not be painted in the webpage. Must be true or false
        // False if not explicitly set here
        "enablePainting" : true,
        // Key of the local storage where the retrieved phone number will be stored for future usages
        "localStorageKey" : "t2cNumber"
    };
    window.t2c = (function () {
        var t2cConfig = {};
        var placeHolderText = "";
        // Create and send XHR request
        var _makeRequest = function (method) {
            var url = _buildRequestURL();
            var request = new XMLHttpRequest();
            return new Promise(function (resolve, reject) {
                request.onreadystatechange = function () {
                    if (request.readyState !== 4) return;
                    if (request.status >= 200 && request.status < 300) {
                        resolve(request.response);
                    } else {
                        reject({
                            status: request.status,
                            statusText: request.statusText
                        });
                    }
                };
                request.open(method || 'POST', url, true);
                request.send();
            });
        };
        var _manageResponse = function(response) {
            _storeNumber(response);
            _updateTargets(response)
        }
        var _manageFailure = function(response) {
            _recoverPlaceholderText();
        }
        var _storeNumber = function(phoneNumber) {
            localStorage[t2cConfig.localStorageKey] = phoneNumber;
            sessionStorage[t2cConfig.localStorageKey] = phoneNumber;
        }
        var _buildRequestURL = function() {
            // TODO build querystring from dict
            var queryStringArguments = {};
            var baseURL = t2cConfig["endpoint"];
            var cookieName = t2cConfig["trackedCookie"];
            var cookieValue = _getCookie(cookieName);
            if (cookieName == "_ga") {
                cookieValue = _parseGACookie(cookieValue);
            }
            var campaignValue = _getCampaignCode();
            var queryString = "?uid=" + cookieValue + "&campaign=" + campaignValue;
            var storedPhone = _retrievePhoneFromLocalStorage();
            if (typeof(storedPhone) === "string") {
                queryString += ("&phone=" + storedPhone)
            }
            var finalURL = baseURL + queryString;
            return finalURL;
        }
        var _getCookie = function(cookieName) {
            var cookieValue = "";
            if (document.cookie.indexOf(cookieName) > -1) {
                cookieValue = document.cookie.split(cookieName + "=")[1].split(";")[0];
            }
            return cookieValue
        }
        // Removes the version string from the GA cookie
        // GA1.2.123456789.87654321 --> 123456789.87654321
        var _parseGACookie = function(cookieValue) {
            var parsedCookie = null;
            if (cookieValue.length > 6) {
                parsedCookie = cookieValue.substr(6);
            }
            return parsedCookie;
        }
        var _getTriggersElements = function() {
            var triggerSelectors = _getConfigArray("triggerSelectors");
            var triggerElements = document.querySelectorAll(triggerSelectors);
            return triggerElements;
        }
        var _updateNumberBooking = function() {
            const sessionNumber = _retrievePhoneFromSessionStorage();
            if (typeof(sessionNumber) !== "string") {
                return;
            }
            var url = t2cConfig.updateBookingEndpoint + "?phone=" + sessionNumber;
            var request = new XMLHttpRequest();
            request.open('POST', url, true);
            request.send();        
        }
        var _attachListener = function(triggerElements, callback) {
            triggerElements.forEach(triggerElement => {
                triggerElement.addEventListener("click", callback);
            });
        }
        var _triggerClickHandler = function() {
            const storedPhone = _retrievePhoneFromSessionStorage();
            if (typeof(storedPhone) === "string") {
                _updateTargets(storedPhone);
            }
            else {
                _hidePlaceholderPhone();
                var request = _makeRequest();
                request.then(_manageResponse, _manageFailure);
            }
        }
        var _retrievePhoneFromLocalStorage = function() {
            return localStorage.getItem(t2cConfig.localStorageKey) || undefined;
        }
        var _retrievePhoneFromSessionStorage = function() {
            return sessionStorage.getItem(t2cConfig.localStorageKey) || undefined;
        }
        var _hidePlaceholderPhone = function() {
            // This is one way of hiding the "old" phone number to avoid flickering
            // This logic can be replaced for any style change such as
            // visibility:hidden, display:none, hiding classes, etc
            if (t2cConfig.enablePainting === false) {
                return;
            }
            placeHolderText = document.querySelector(_getConfigArray("targetSelectors")[0]).innerHTML;
            _updateTargets("");
        }
        var _recoverPlaceholderText = function() {
            if (t2cConfig.enablePainting === false) {
                return;
            }
            _updateTargets(placeHolderText);
        }
        var _updateTargets = function(HTMLcontent) {
            if (t2cConfig.enablePainting === false) {
                return;
            }
            var targetSelectors = _getConfigArray("targetSelectors");
            var targetElements = document.querySelectorAll(targetSelectors);
            targetElements.forEach(element => {
                element.innerHTML = HTMLcontent;
            });
            var targetHrefSelectors = _getConfigArray("targetHrefSelectors");
            var targetHrefElements = document.querySelectorAll(targetHrefSelectors);
            targetHrefElements.forEach(element => {
                element.href = "tel:".concat(HTMLcontent);
            });
        }
        var _getCampaignCode = function() {
            var campaignCode = _getCountryCodeForCampaign();
            return campaignCode;
        }
        var _getCountryCodeForCampaign = function() {
            var countryCode = "";
            dataLayer.forEach(dlElement => {
                if(dlElement.paisGeoIp !== undefined) countryCode = dlElement.paisGeoIp;
            })
            return countryCode;
        }
        var _getConfigArray = function(identifier) {
            var configArray = t2cConfig[identifier];
            if(typeof(configArray) === "string") {
                configArray = [configArray];
            }
            return configArray;
        }
        // Actual initialization function
        var _init = function (config) {
            t2cConfig = config;
            var triggerElements = _getTriggersElements();
            _updateNumberBooking();
            _attachListener(triggerElements, _triggerClickHandler);
        }
        /* Main interface - everything here will be public
        */
        var core = {
            init : function (config) {
                // Call real init function
                _init(config);
            }, 
            version : function () {
                console.log("v0.5.0 2021-03-05");
            },
            __debug__ : {
                respond : function(number) {
                    _updateTargets(number);
                }
            },
        }
        return core;
    }());
    /* 
        Initialize library
    */
    window.t2c.init(config_t2c_tag);
}   
<!-- The page was last published on 12/01/2022 16:30:00-->
<!-- The page was last published on 12/01/2022 16:59:56-->