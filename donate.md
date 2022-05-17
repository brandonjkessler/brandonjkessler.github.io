<nav>
{% assign pages = site.pages %}
{% for page in pages %}
  <li><a href="{{page.url}}">{{page.title}}</a></li>
{% endfor %}
</nav>

## Donations

If you appreciate what I provide on my github page or if you find my blog useful please buy me some caffeine!

### PayPal

<div id="donate-button-container">
<div id="donate-button"></div>
<script src="https://www.paypalobjects.com/donate/sdk/donate-sdk.js" charset="UTF-8"></script>
<script>
PayPal.Donation.Button({
env:'production',
hosted_button_id:'ZMTNEAA47P6XG',
image: {
src:'https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif',
alt:'Donate with PayPal button',
title:'PayPal - The safer, easier way to pay online!',
}
}).render('#donate-button');
</script>
</div>


### Crypto

