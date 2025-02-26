---
layout: post
# title:  "Powerful things you can do with the Markdown editor"
author: sal
categories: [ Jekyll, tutorial ]
image: assets/images/1.jpg
---
{% assign BITS = 8 %}
<html>
<head>
</head>
<body>
  <img src="https://j.gifs.com/YE6OJA.gif" alt="Animated GIF">
</body>
</html>
<style>
    td {
        text-align: center;
        vertical-align: middle;
    }
</style>

<br>

{% comment %}
Liquid for loop includes last number, thus the Minus
{% endcomment %}
{% assign bits = BITS | minus: 1 %} 

<table>
    <thead>
        <tr>
            {% comment %}
            Build many bits
            {% endcomment %}
            {% for i in (0..bits) %}
            <th><img id="bulb{{ i }}" src="{{site.baseurl}}/assets/images/bulb_off.png" alt="" width="40" height="Auto">
                <div class="button" id="butt{{ i }}" onclick="javascript:toggleBit({{ i }})">Turn on</div>
            </th>
            {% endfor %}
        </tr>
    </thead>
    <tbody>
        <tr>
            {% comment %}
            Value of bit
            {% endcomment %}
            {% for i in (0..bits) %}
            <td><input type='text' id="digit{{ i }}" Value="0" size="1" readonly></td>
            {% endfor %}
        </tr>
    </tbody>
</table>



<button id = "submit">Convert</button>


<script>

    // header := w.Header()
    // header.Add("Access-Control-Allow-Origin", "*")

    // if r.Method == "OPTIONS" {
    //     w.WriteHeader(http.StatusOK)
    //     return
    // }   

    const BITS = {{ BITS }};
    const MAX = 2 ** BITS - 1;
    const MSG_ON = "Turn on";
    const IMAGE_ON = "{{site.baseurl}}/assets/images/bulb_on.gif";
    const MSG_OFF = "Turn off";
    const IMAGE_OFF = "{{site.baseurl}}/assets/images/bulb_off.png";
    const submit = document.getElementById("submit");
    submit.addEventListener("click", send);


    // return string with current value of each bit
    function getBits() {
        let bits = "";
        for(let i = 0; i < BITS; i++) {
            bits = bits + document.getElementById('digit' + i).value;
        }
        return bits;
    }
    // setter for Document Object Model (DOM) values
    function setConversions(binary) {
        document.getElementById('binary').innerHTML = binary;
        // Octal conversion
        document.getElementById('octal').innerHTML = parseInt(binary, 2).toString(8);
        // Hexadecimal conversion
        document.getElementById('hexadecimal').innerHTML = parseInt(binary, 2).toString(16);
        // Decimal conversion
        document.getElementById('decimal').innerHTML = parseInt(binary, 2).toString();
    }
    // convert decimal to base 2 using modulo with divide method
    function decimal_2_base(decimal, base) {
        let conversion = "";
        // loop to convert to base
        do {
            let digit = decimal % base;           // obtain right most digit
            conversion = "" + digit + conversion; // what does this do? inserts digit to front of string
            decimal = ~~(decimal / base);         // what does this do? divides by base what is ~~? force whole number
        } while (decimal > 0);                    // why while at the end? 0 pads front of binary number
            // loop to pad with zeros
            if (base === 2) {                     // only pad for binary conversions
                for (let i = 0; conversion.length < BITS; i++) {
                    conversion = "0" + conversion;
            }
        }
        return conversion;
    }
    // toggle selected bit and recalculate
    function toggleBit(i) {
        //alert("Digit action: " + i );
        const dig = document.getElementById('digit' + i);
        const image = document.getElementById('bulb' + i);
        const butt = document.getElementById('butt' + i);
        // Change digit and visual
        if (image.src.match(IMAGE_ON)) {
            dig.value = 0;
            image.src = IMAGE_OFF;
            butt.innerHTML = MSG_ON;
        } else {
            dig.value = 1;
            image.src = IMAGE_ON;
            butt.innerHTML = MSG_OFF;
        }
        // Binary numbers
        const binary = getBits();
        setConversions(binary);
    }

    // add is positive integer, subtract is negative integer
    function add(n) {
        let binary = getBits();
        // convert to decimal and do math
        let decimal = parseInt(binary, 2);
        if (n > 0) {  // PLUS
            decimal = MAX === decimal ? 0 : decimal += n; // OVERFLOW or PLUS
        } else  {     // MINUS
            decimal = 0 === decimal ? MAX : decimal += n; // OVERFLOW or MINUS
        }
        // convert the result back to binary
        binary = decimal_2_base(decimal, 2);
        // update conversions
        setConversions(binary);
        // update bits
        for (let i = 0; i < binary.length; i++) {
            let digit = binary.substr(i, 1);
            document.getElementById('digit' + i).value = digit;
            if (digit === "1") {
                document.getElementById('bulb' + i).src = IMAGE_ON;
                document.getElementById('butt' + i).innerHTML = MSG_OFF;
            } else {
                document.getElementById('bulb' + i).src = IMAGE_OFF;
                document.getElementById('butt' + i).innerHTML = MSG_ON;
            }
        }
    }
    
    //sends number and data to backend
function send() {
    var binary = getBits();
        var myHeaders = new Headers();
        const url = "http://127.0.0.1:8086/api/binary/post";
    const headers = {
        method: 'GET', // *GET, POST, PUT, DELETE, etc.
        mode: 'no-cors', // no-cors, *cors, same-origin
        cache: 'default', // *default, no-cache, reload, force-cache, only-if-cached
        credentials: 'omit', // include, *same-origin, omit
        headers: {
        'Content-Type': 'application/json'
        // 'Content-Type': 'application/x-www-form-urlencoded',
        },
    };
        // myHeaders.append("Content-Type", "application/json");
        // myHeaders.append("Accept", "application/json");
        // myHeaders.append("Origin", "http://127.0.0.1:4000/mediumish-theme-jekyll/powerful-things-markdown-editor/");
    var raw = JSON.stringify({
        "tag": binary
    });

    var requestOptions = {
        method: 'POST',
        headers: myHeaders,
        body: raw,
        redirect: 'follow'
    };

    fetch(url, requestOptions)
        .then(response => response.json())
        .then(result => console.log(result))
        .catch(error => console.log('error', error));
}







    // fetch("http://127.0.0.1:8086/api/y/query", {
    // method: "POST",
    // body: JSON.stringify({
    //     query: binary
    //     title: "Binary Data",
    //     completed: false
    // }),
    // headers: {
    //     "Content-type": "application/json; charset=UTF-8"
    // }
    // })
    // .then((response) => response.json())
    // .then((json) => console.log(json));

    // const xhr = new XMLHttpRequest();
    // const url = 'http://127.0.0.1:8086/api/y/<string:query>'; 

    // xhr.open('POST', url, true);
    // xhr.setRequestHeader('Content-Type', 'application/json');

    // xhr.onreadystatechange = function() {
    //     if (xhr.readyState === 4 && xhr.status === 200) {
    //       console.log('Data sent successfully');
    //     // Handle the response from the backend if needed
    //     }
    // };
    // console.log(binary);
    // const data = JSON.stringify({ binary : binary });
    // xhr.send(data);
    
function send() {
  var binary = getBits();
  console.log(binary);
  const url = "http://127.0.0.1:8086/api/binary/post";
  const headers = {
      'Content-Type': 'application/json'
  };

  var raw = JSON.stringify({
    "tag": binary
  });

  var requestOptions = {
    method: 'POST',
    headers: headers,
    body: raw,
    mode: 'cors', // Set the mode directly in requestOptions
    redirect: 'follow',
    cache: 'default',
    credentials: 'omit'
  };

  fetch(url, requestOptions)
    .then(response => response.json())
    .then(result => console.log(result))
    .catch(error => console.log('error', error));
}

</script>
