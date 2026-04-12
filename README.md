# Chatbase

Realtime chat using GraphQL Live Queries, Next.js and NextAuth.js &mdash; [tutorial](https://grafbase.com/guides/how-to-build-a-real-time-chat-app-with-nextjs-graphql-and-server-sent-events)

## Tools used

- NextAuth.js
- Next.js
- Apollo Client
- Grafbase
- Server-Sent Events
- GraphQL Live Queries
- GraphQL
- Tailwind CSS

## Local Development

1. `npm install`
2. Create a [GitHub OAuth App](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app) with your app details for development purposes. Make sure to set `Authorization callback URL` to `http://localhost:3000/api/auth/callback/github`
3. `cp .env.example .env` and add values for `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` from step 2.
4. [Generate a secret value](https://generate-secret.vercel.app) for `NEXTAUTH_SECRET` and add it to `.env`
5. `cp grafbase/.env.example grafbase/.env`
6. Add the same `NEXTAUTH_SECRET` to `grafbase/.env`
7. `npx grafbase dev`
8. `npm run dev`

## Deploy to Production

1. Fork and Push this repo to GitHub
2. [Create an account](https://grafbase.com) with Grafbase
3. Create new project with Grafbase and connect your forked repo
4. Add environment variable `NEXTAUTH_SECRET` during project creation
5. Create a [GitHub OAuth App](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app) with your app details for production purposes. Make sure to set `Authorization callback URL` to `[YOUR_DESIRED_VERCEL_DOMAIN]/api/auth/callback/github`
6. Deploy to Vercel and add `.env` values (`NEXT_PUBLIC_GRAFBASE_API_URL`\*, `NEXTAUTH_SECRET`, `GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`)

\* `NEXT_PUBLIC_GRAFBASE_URL` is your production API endpoint. You can find this from the **Connect** modal in your [project dashboard](https://grafbase.com/dashboard).

`jdbc:databricks://<host>:443/default;httpPath=<path>;AuthMech=3;EnableBatchedInserts=1;BatchInsertSize=1000;supportManyParameters=1;`

%dw 2.0
output application/csv
    separator     = "|",       // pipe as delimiter
    quoteValues   = true,      // wrap every cell in quotes so embedded pipes are safe
    encoding      = "UTF-8",   // explicit UTF-8 for S3 and Databricks compatibility
    header        = true,      // first row is column names derived from payload keys
    lineSeparator = "\n",      // Unix LF line endings expected by Databricks Auto Loader
    escape        = '"'        // RFC-4180 quote escaping aligned with DWL 1 clean output

---
payload                        // clean JSON from DWL 1 passed straight to CSV writer


%dw 2.0
output application/json

fun clean(v: Any): String =
    (v default "") as String                // if value is null, default to empty string then cast to String
    replace /<[^>]+>/   with " "           // find anything between < and > tags and replace with space
    replace /&amp;/     with "&"           // find the literal text &amp; and replace with & symbol
    replace /&nbsp;/    with " "           // find the literal text &nbsp; and replace with a space
    replace /&quot;/    with '"'           // find the literal text &quot; and replace with a double quote
    replace /\\/        with "\\\\"        // find any backslash and escape it by doubling it
    replace /\r\n/      with " "           // find carriage return followed by newline and replace with space
    replace /[\r\n\t]/  with " "           // find any lone carriage return, newline or tab and replace with space
    replace /[ ]{2,}/   with " "           // find two or more consecutive spaces and collapse into one space
    then trim($)                           // take the result and remove any leading or trailing spaces

---
payload map (r) ->                         // loop through each record in the Salesforce response array
    r mapObject ((value, key) -> {         // loop through every key value pair in the current record
        (key): if (value == null)        null          // if the field has no value write null as-is
               else if (value is String) clean(value)  // if the field is text run it through the sanitiser
               else                      value         // if the field is a number boolean or date leave it untouched
    })
