Step 1 — Run the database migration in Supabase

Go to supabase.com → open your project
Click SQL Editor in the left sidebar
Click New query
Open the migration.sql file from your outputs folder, copy all the text
Paste it into the SQL editor and click Run
You should see "Success" — this creates the new tables and adds Client X with 10 credits


Step 2 — Update your main app (adobe-status)
Open your terminal and go to your adobe-status folder:
cd path/to/adobe-status
(login details removed)
npm install resend
Now copy the files from main-app-updates/ into the same paths inside adobe-status. For example:

main-app-updates/app/admin/dashboard/page.js → paste over adobe-status/app/admin/dashboard/page.js
main-app-updates/app/api/admin/requests/route.js → create that folder path and paste the file in
Same for revoke/, clients/, logs/

Then open your .env.local file in the adobe-status folder and add these 3 lines (from .env.additions):
RESEND_API_KEY=(secret, removed)
(login details removed)
NEXT_PUBLIC_STATUS_URL=http://localhost:3000
Get your Resend API key free at resend.com → sign up → API Keys → Create Key.
Now run the main app:
npm run dev
Open http://localhost:3000/admin — log in with admin@big and you'll see the new tabs (Pending Requests, Revoke Requests, Client Accounts, Activity Log).

Step 3 — Run the Client X dashboard
Open a second terminal window. Go to the client-x-dashboard folder from your outputs:
cd path/to/outputs/client-x-dashboard
Install dependencies:
npm install
Create a .env.local file in that folder (copy the same Supabase values from your main app):
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
Run it on a different port so both apps run at the same time:
npm run dev -- -p 3001
Open http://localhost:3001 — log in with clientx@bigmembres.com / clientx@123

How to test the full flow:

In Client X dashboard (port 3001) → Add User tab → fill in a name, email, plan → Submit Request (costs 1 credit)
Switch to your main admin dashboard (port 3000/admin) → Pending Requests tab → you'll see it appear → click Approve
The user gets an activation email, and they can now check their status on the public page
Back in Client X (port 3001) → My Requests tab → you'll see the 12hr countdown timer start
If you want to test revoke, click Revoke in Client X → go back to your admin → Revoke Requests tab → Approve it → credit comes back
