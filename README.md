How to Use the Bookmarklet to Extract Local Storage
Create the Bookmarklet:

Copy this code:
javascript:(function() { const data = JSON.stringify(localStorage); console.log(data); alert('Local Storage data logged to console.'); })();
Add to Favorites Bar:

Right-click the bookmarks bar, select "Add Page," name it, and paste the code in the "URL" field.
Show Hidden Bookmark Bar (if needed):

Chrome: Press Ctrl+Shift+B (Windows/Linux) or Cmd+Shift+B (macOS) to show the bookmarks bar.
Firefox: Go to the menu, select "Customize Toolbar," and drag the "Bookmarks Toolbar" to make it visible.
Run the Bookmarklet:

Visit the page, click the bookmarklet, and check the console (Ctrl+Shift+I or F12) for local storage data.
Let me know if you'd like to tweak anything!
