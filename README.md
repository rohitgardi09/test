
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Indian Flag</title>

<style>
body{
    margin:0;
    height:100vh;
    display:flex;
    justify-content:center;
    align-items:center;
    background:#f5f5f5;
}

.flag{
    width:450px;
    height:300px;
    box-shadow:0 0 10px rgba(0,0,0,0.2);
}

.saffron{
    height:100px;
    background:#FF9933;
}

.white{
    height:100px;
    background:#FFFFFF;
    position:relative;
}

.green{
    height:100px;
    background:#138808;
}

.chakra{
    position:absolute;
    top:50%;
    left:50%;
    transform:translate(-50%, -50%);
    width:80px;
    height:80px;
    border:3px solid #000080;
    border-radius:50%;
}

.spoke{
    position:absolute;
    width:2px;
    height:40px;
    background:#000080;
    top:0;
    left:50%;
    transform-origin:bottom center;
}
</style>
</head>

<body>

<div class="flag">
    <div class="saffron"></div>

    <div class="white">
        <div class="chakra" id="chakra"></div>
    </div>

    <div class="green"></div>
</div>

<script>
const chakra = document.getElementById("chakra");

for(let i = 0; i < 24; i++) {
    const spoke = document.createElement("div");
    spoke.className = "spoke";
    spoke.style.transform =
        `translateX(-50%) rotate(${i * 15}deg)`;
    chakra.appendChild(spoke);
}
</script>

</body>
</html>
