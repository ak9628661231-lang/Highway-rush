
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width,initial-scale=1">
<style>
*{box-sizing:border-box}
body{margin:0;background:#25852e;overflow:hidden}
.game{height:100dvh;display:flex;justify-content:center}
.road{width:100%;max-width:430px;height:100%;background:#555;position:relative;border-left:7px solid white;border-right:7px solid white;overflow:hidden}
.line{position:absolute;top:-82px;bottom:-82px;width:6px;background:repeating-linear-gradient(to bottom,white 0 42px,transparent 42px 82px);animation:move .65s linear infinite}
.a{left:33.333%}
.b{left:66.666%}
@keyframes move{to{background-position:0 82px}}
</style>
</head>
<body>
<div class="game">
<div class="road">
<div class="line a"></div>
<div class="line b"></div>
</div>
</div>
</body>
</html>
