// Function to get all items from localStorage
function getLocalStorage() {
    let localStorageData = {};
    for (let i = 0; i < localStorage.length; i++) {
        let key = localStorage.key(i);
        localStorageData[key] = localStorage.getItem(key)
;
    }
    return localStorageData;
}

// Function to get all items from sessionStorage
function getSessionStorage() {
    let sessionStorageData = {};
    for (let i = 0; i < sessionStorage.length; i++) {
        let key = sessionStorage.key(i);
        sessionStorageData[key] = sessionStorage.getItem(key)
;
    }
    return sessionStorageData;
}

// Function to get all cookies
function getCookies() {
    let cookies = document.cookie.split('; ');
    let cookieData = {};
    cookies.forEach(cookie => {
        let [name, value] = cookie.split('=');
        cookieData[name] = value;
    });
    return cookieData;
}

// Function to export data as JSON file
function exportStorageDataAsJSON() {
    const data = {
        localStorage: getLocalStorage(),
        sessionStorage: getSessionStorage(),
        cookies: getCookies()
    };
    const json = JSON.stringify(data, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'storageData.json';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
}

// Call the function to export storage data as JSON file
exportStorageDataAsJSON();
