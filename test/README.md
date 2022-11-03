aaa2
<script>
  //window.location.href='okex://metaX/dex/swap';
  
  if (window.webkit) {
      var url = { "uri": "window", "method": "nav", "data": "main/spot/buy", "success": "success", "fail": "fail" };
      window.webkit.messageHandlers.JShandle.postMessage(url);
      var close = { "uri": "window", "method": "close", "data": true }
      window.webkit.messageHandlers.JShandle.postMessage(close);
  }
</script>

TEST jump8
