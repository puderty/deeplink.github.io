aaa5
<script>
function isiOSDevice() {
        if (navigator.userAgent.indexOf('iPhone')>-1 || navigator.userAgent.indexOf('iOS')>-1) {
            return true;
        } else {
            return false;
        }
    }
function jsbridge(msg) {
        if (true) {
        alert("ios1")
            if (window.webkit) {
        alert("ios")
                window.webkit.messageHandlers.JShandle.postMessage(msg);
            }
        } else {
            var msgStr = JSON.stringify(msg);
            prompt(msgStr);
        }
    }
window.location.href='okex://metaX/dex/swap';
jsbridge({"uri":"window","method":"close","data":true});
</script>

TEST jump9
