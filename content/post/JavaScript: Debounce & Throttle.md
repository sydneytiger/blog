Event handling in browser costs. Events like **keyup**, **scroll** can be triggered multiple times within a very short period of time. When a complex function is subscribed to these kind of events, 

```html
<!DOCTYPE html>
<html>
<head>
    <title>Keyup event</title>
</head>
<body>
    <div>
        keyup event：<input type="text" id="index"/>
    </div>
    <script>
        window.onload = () => {
            function fakeAjax (data) {
                console.log(new Date().toLocaleTimeString() + ' - ' + data)
            }

            document.querySelector('#index').addEventListener('keyup', e => {
                fakeAjax(e.target.value)
            })
        }
    </script>
</body>
</html>
```