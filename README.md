# Hi there 👋

## Bookmarklets

[Search LDFF](javascript:function(){let e=document.querySelectorAll('[aria-label$="to clipboard"][aria-label^="Copy"]')[0]?.previousSibling.innerHTML;if(e||(e=prompt("What's the LaunchDarkly FF Key?")),!e)return;const a=e.replaceAll("-",""),l="https://github.com/search?type=code&q=org%3Asamcart+%28"+e+"+OR+"+a+"%29";window.open(l,"_blank").focus()}();)