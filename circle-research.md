*   I asked Justin specifically if we what we care about right now is just Circle or if it also includes our custom CreatorU app. He said right now we only care about integrating Circle and to not worry about our custom app. My hope is that our custom CreatorU app will get retired before we need more sophisticated integration with it than we have now. But we'll see I guess.
    
*   Circle is built with white labeling in mind. I think JC shared this general white labeling page with me before: [https://help.circle.so/p/circle-plus/implementation-and-launch/white-labeling-various-aspects-of-your-community](https://help.circle.so/p/circle-plus/implementation-and-launch/white-labeling-various-aspects-of-your-community)
    
*   All our current Circle stuff is set up through SamCart integrations and product redirects. You can see the current integrations we have set up here: [https://app.samcart.com/marketplace/checkout/apps/manage/circle](https://app.samcart.com/marketplace/checkout/apps/manage/circle)
    
*   We use product redirects after CreatorU products to redirect customers to a Circle sign up page. It's not great because with the integrations we only connect between SamCart and Circle by email. If they sign up with Circle using a different email than they used to purchase SamCart or if they change their email in either system then the integrations get messed up. It's not perfect, but it's worked "well enough" for now.
    
*   We use the custom domain [members.creatoru.com](http://members.creatoru.com/) for our production Circle instance.
    
*   We have a test Circle instance. There are creds for it named "Circle Test Account" in 1Pass. The test instance is not set up at **all** like the production instance.
    
*   If you need access to the production Circle instance (which eventually we will), Sara can hook you up.
    
*   Circle provides at least 2 different APIs, what they call the "Admin API" and the "Headless" API. Most of what Dave and I have looked at was with the Headless API.
    
*   You can create different types of API tokens for the different types of APIs. So far I've only created a Headless API token and played with that. I think you need an admin account to create any API token. That's relatively easy to do in our test Circle account. It's a little more complicated in our production account (just because of license limits and the like).
    
*   The main documentation for the Headless API is here: [https://api.circle.so/apis/headless/quick-start](https://api.circle.so/apis/headless/quick-start)
    
*   There's this specific documentation page about using the Headless API to sign people in, either in a new tab or an iframe: [https://api.circle.so/apis/headless/member-api/cookies#injecting-session-cookies-via-url-redirect](https://api.circle.so/apis/headless/member-api/cookies#injecting-session-cookies-via-url-redirect). Dave and I experimented with the "Injecting session cookies via URL redirect" option. It might be worth looking at the "Injecting session cookies via API" option, but we thought that looked more complicated for our use case.
    
*   The Headless API lets you send your API token to [https://app.circle.so/api/v1/headless/auth_token](https://app.circle.so/api/v1/headless/auth_token) and get back an access token to sign in as a member of your community. The docs for that endpoint are at [https://api.circle.so/apis/headless/quick-start#auth-api](https://api.circle.so/apis/headless/quick-start#auth-api). Here's the Restbook command I used to call that endpoint:
    

```curl
POST https://app.circle.so/api/v1/headless/auth_token  Authorization: Bearer {Test account Headless API token}  Content-Type: application/json  {    "email": "jtravis+20250114143@samcart.com"  }   
```

*   The response from that API call looks something like this:
    

```   
{"refresh_token":"J0ZS_jVPXHawt6PAXG2qv-6hbwzm8Hgr9dShbaN4tfQ","refresh_token_expires_at":"2025-09-25T21:44:02.000Z","community_member_id":31815693,"community_id":126930,"access_token":"eyJhbGciOiJSUzI1NiJ9.eyJleHAiOjE3NTY4NDkzOTcsInR5cGUiOiJoZWFkbGVzc19tZW1iZXJfdG9rZW4iLCJqdGkiOiJiMGNkN2I5ZS1jYmMxLTRmYmEtYjc3OS1jMzE1YzY1MGYzOTUiLCJjb21tdW5pdHlfaWQiOjEyNjkzMCwiY29tbXVuaXR5X21lbWJlcl9pZCI6MzE4MTU2OTN9.biwKbwQSJPbjm1eS5o5CMXfpnAGLtmqALWU2X2T_GHA_UOn63XCzwVaDuQ32mIIAU5iqTTYsSAXmF7Kel9MI2YwImpcVd5I6wSh_sVOchIj7aHrRF6FfI_0TdO9IQFOtqy1LyoaxOcZ2K5sOmdH7LcgZQ5GL7Edpb9-53lAf3VEjnrqIegXqrRLj8eM0YxBNHfRBA6wVSAFnI2lp63EYHhUp8NkYgX2OiX-IW2VwRQobkFao9WQOiXJgPuuZsbD0tMzu1VrEszilP9e-LYWQvozHslVGGlCbWRJDpygZEze-QSWsSwAuyNWFZVLD4VgCfTf30wfJsJ5YGlXBo6NWqQ","access_token_expires_at":"2025-09-02T21:43:17.620Z"}   
```

*   You can take the access_token from that response and build a URL that will redirect and sign in a user. The URL will look like this: [https://sync-tank.circle.so/session/cookies?access_token={access](https://sync-tank.circle.so/session/cookies?access_token={access) token}.
    
*   The docs for how to construct this URL are at [https://api.circle.so/apis/headless/member-api/cookies#injecting-session-cookies-via-url-redirect](https://api.circle.so/apis/headless/member-api/cookies#injecting-session-cookies-via-url-redirect).
    
*   sync-tank is the name of our testing Circle account.
    
*   If you go that URL (either in a new tab or use it as the src of an iframe), you'll get signed in as the user and redirected to the main Circle page for that community.
    
*   When requesting an access token for a user, you can search by email, sso_id, or community_member_id. Right now the only thing we have to search by is email.
    
*   The access token in the API response is good for an hour.
    
*   Once you are signed in using the constructed URL above, I think the main auth cookie for Circle is named _circle_session. That cookie is a session cookie. I'm a little fuzzy on exactly how session cookies work with iframes, so that might need additional investigation.
    
*   In order to get all of this to work, we'd need to generate an API token for our production Circle instance, store that in an env var in foundation, and create an endpoint that uses that API token to generate a Circle sign in URL for the current user. Or something like that at least.
    
*   As mentioned above, we currently create new Circle users through a SamCart integration. However, in the circle admin API there is an endpoint for creating/inviting new users. Theoretically we could write some code to call this endpoint ourselves and have a much tighter, less error prone integration with Circle.
    
*   The docs for this API endpoint are here: [https://api-headless.circle.so/?urls.primaryName=Admin API V2#/Community Members/post_api_admin_v2_community_members](https://api-headless.circle.so/?urls.primaryName=Admin API V2#/Community Members/post_api_admin_v2_community_members)
    
*   If we used this option for creating users I think we can create their credentials so they wouldn't have to manually sign up and pick a password like they do now. I didn't try calling this API though, so I'm not 100% sure on that.
    
*   If we created users with this API we might also get a better ID we could store. In that case we might not have to get an access token by email, which would make the system less brittle (email changes on the SamCart or Circle side would break the link between the two).
    
*   Calling the admin API to create users instead of using the current integrations would be a fairly significant lift though. We currently use several different integrations on different products to determine what exactly customers get access to in Circle. To replace that we'd probably need to introduct some additional product metadata or something we could use to determine when to create a Circle user on product purchase.
    
*   Given the additional work it would take to call the Circle create/invite user correctly, I think it would probably make sense to do that in a future interation and just rely on our existing, but less idea flow using integrations and product redirects to create the user in Circle. But it's possible there's an easier solution to call the admin API correctly than what I thought of.
    
*   Most of the customer access in Circle is managed using Space Groups in Circle, though Sara said there are some special one offs too.
    
*   Sara also mentioned that currently when people sign up for SamCart and CreatorU they have two separate subscriptions that we sell using a bundled product. The can cancel the two subscriptions independently. Doing that same thing seems complicated for Creator Studio. I assume for Creator Studio we'd have a single subscription that grants access to all three systems (SamCart, TypeSet, and CreatorU). I think that would work, but it might be worth additional discovery and experimentation.