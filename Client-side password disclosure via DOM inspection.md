It can happen during a pentest that a seemingly protected password field is only masked on the client side, while the actual value remains accessible within the DOM. This repository demonstrates a simple yet illustrative scenario where a prefilled password, hidden behind type="password", can be exposed by inspecting and modifying the HTML through browser developer tools.

![Example 1: password field in DOM](images/example1.png)
