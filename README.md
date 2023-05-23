pm.test("Show Toast Alert", function () {
    pm.visualizer.showToast({
        title: "Test Alert",
        message: "This is a toast alert message",
        position: "topRight",
        type: "info", // can be "success", "info", "warning", or "error"
        displayMilliseconds: 5000 // duration in milliseconds, set as per your requirement
    });
});
