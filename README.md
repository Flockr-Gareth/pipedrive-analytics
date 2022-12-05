# Pipedrive Email Analytics



Pipedrive is a fantastic tool for CRM, but is lacking in some of the basic needs for email analytics outside of the pipedrive application.

Currently it does not offer the following: -

1. UTM parameters for standard (GA) link tracking
2. UserID setting in GA (https://support.google.com/analytics/answer/9213390?hl=en)
3. Linking website visitor tracking to Pipedrive



This solution attempts to solve these problems using workarounds and commonly available tools.

You will need a Zapier account (free trial available) and GA4 (UA is supported but google are ending it's life in June 2023)



## Step 1 Custom Fields

In order to use merge tags in emails for links (more on this later) you will need to add 2 custom fields to the Person object in Pipedrive, add the following

(note: system fields such as the ID of a person are not available is merge tags, so we use this work around)

1. PIPEDRIVE_ID (type: Numeric)
2. UTM_CAMPAIGN (type: Text)



## Step 2 Tracking Script

Add the following script to the header of your website, or use a tag manager, orr however you add tags

```html
<script>
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
let usp = new URLSearchParams(location.search);
	if(usp.has('pid')) {
gtag('config', 'G-15ESTEYQMW', {'user_id': usp.get('pid')});
$.get( "https://hooks.zapier.com/hooks/catch/<your acc>/<your zap>/", function( data ) {}); }
</script>
```

This will do 2 things, first it looks for the querystring parameter ?pid (more on this later), if found in the URL it will tell GA what the userID is (the pipedrive ID for this Person), secondly it will call a Zap to log the time of the website visit (again more below)

## Step 3 Zapier

Head over to https://zapier.com/ and create an account, I have shared 3 zap templates that you will need

1. https://zapier.com/shared/c98e563a0de669f93da1030fd5b1ee37b07e268e this Zap runs on the hour and loops through all the Persons in your Pipedrive account, finds their Pipedrive ID and sets a the custom field PIPEDRIVE_ID with the value (so you can use it in a merge tag)
   1. You will need to update the Zaps with your account, it will ask for permissions - the Zap runs in your account, this is just a template
   2. You will need to refresh the fields and remove th eunmatched field and select YOUR custom field PIPEDRIVE_ID which you created in step 1, for its value set the ID of the Person from the Find Person step in the Zap
   3. Once this has run and you have confirmed the field has been populated for all Persons, you can disable the Zap - it runs on the hour (there are ways to make a Zap run manually but its a bit conveluted)
2. https://zapier.com/shared/16cd8b8dbdaea40fa1b50eae28acedff386e661d
   1. This Zap runs whenever a new Person is added to Pipedrive, it sets the custom field PIPEDRIVE_ID with the Persons ID from Pipedrive
3. https://zapier.com/shared/355b6b697ad2287ad0e2164bf0c6cb076ff52862
   1. This Zap is called by the script in Step 2 (on your website) to set the Last visit time of a person who visits your website
   2. You will need to update the URL "https://hooks.zapier.com/hooks/catch/<your acc>/<your zap>/" with the value of the first step in the Zap (you will have your own URL)



## Step 4 Emails

For the final step, for any email that you send out, add this following to the URL of any link: -

`?utm_medium=EMAIL&utm_source=Pipedrive&utm_campaign=*|UTM_CAMPAIGN|*&pid=*|PIPEDRIVE_ID|*`

e.g.

`https://flockr.co/?utm_medium=EMAIL&utm_source=Pipedrive&utm_campaign=*|UTM_CAMPAIGN|*&pid=*|PIPEDRIVE_ID|*`

if your link already has a querystring (has a ? in it), append with a & instead, e.g. add

`&utm_medium=EMAIL&utm_source=Pipedrive&utm_campaign=*|UTM_CAMPAIGN|*&pid=*|PIPEDRIVE_ID|*`



## Final Thoughts

Until Pipedrive add this as standard this workaround should work.

You will be able to view what pages and events Persons in Pipedrive do on yourwebsite by matching up the User ID in the reports on GA to the Pipedrive ID.

Note: GA does not allow the storing of PII in GA, this is why we use the Pipedrive ID instead of email or something identifiable.
