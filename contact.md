---
layout: page
title: Contact
---

<script>
function say_thanks(){
email = document.getElementsByName('_replyto')[0].value;
if(email === ''){email="Mr. X"}
alert("Thank you, "+email);
}
</script>
<p>Have a question? A comment? A joke? You can find me as <a href="http://twitter.com/pranavek">@pranavek</a> on twitter or just use the contact form below:</p>
<form action="https://formspree.io/hello@pranavek.com" method="POST" name="contactForm" onsubmit="say_thanks()">

<input type="hidden" name="_next" value="http://blog.pranavek.com">

<input id="subject" type="text" name="subject" placeholder="Subject (optional)" ><br>

<input id="email" type="email" name="_replyto" placeholder="Email Address, so I can get back to you" ><br>

<textarea required id="msg" name="message" style="min-width:100%;height:150px;" placeholder="Your Interesting Message" ></textarea>

<div class="center-text">

<button type="submit">Send Message&raquo;</button>

</div>

</form>
