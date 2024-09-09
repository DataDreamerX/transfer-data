// Function to get all items from localStorage
function getLocalStorage() {
    let localStorageData = [];
    for (let i = 0; i < localStorage.length; i++) {
        let key = localStorage.key(i);
        localStorageData.push({ name: key, value: localStorage.getItem(key) });
    }
    return localStorageData;
}

// Function to get all items from sessionStorage
function getSessionStorage() {
    let sessionStorageData = [];
    for (let i = 0; i < sessionStorage.length; i++) {
        let key = sessionStorage.key(i);
        sessionStorageData.push({ name: key, value: sessionStorage.getItem(key) });
    }
    return sessionStorageData;
}

// Function to get all cookies
function getCookies() {
    let cookies = document.cookie.split('; ');
    let cookieData = [];
    cookies.forEach(cookie => {
        let [name, value] = cookie.split('=');
        cookieData.push({
            name: name,
            value: value,
            domain: document.domain,
            path: '/',
            expires: -1,
            httpOnly: false,
            secure: false,
            sameSite: 'Lax'
        });
    });
    return cookieData;
}

// Function to export data as JSON file
function exportStorageDataAsJSON() {
    const data = {
        cookies: getCookies(),
        origins: [
            {
                origin: window.location.origin,
                localStorage: getLocalStorage(),
                sessionStorage: getSessionStorage()
            }
        ]
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
