# Invoice Chaser: deploy guide

Freelancers add invoices; the app emails clients polite, escalating reminders
(3 days before due, then 1, 7, 14 and 21 days after) until paid.
Free plan: 3 invoices, copy-paste messages. Paid plan ($9/mo): unlimited
invoices, one-click sending and automatic follow-ups.

## Run it locally (Node 18+)
    npm install
    cp .env.example .env     # fill in the values
    npm start                # http://localhost:3000

## Launch checklist
1. **Resend (email):** create an account, verify a sending domain, create an
   API key. Put it in RESEND_API_KEY, and use an address on that domain as FROM_EMAIL.
   Replies go to your customer, who is also BCC'd on every message.
2. **Stripe (payments):** create a product with a $9/month recurring price and
   copy its price ID into STRIPE_PRICE_ID. Copy your secret key into STRIPE_SECRET_KEY.
   Enable the Customer Portal in Stripe settings so people can cancel.
3. **Host (Render, Railway or Fly):** create a Node web service from your Git repo.
   Build: `npm install`. Start: `npm start`. Attach a **persistent disk/volume**
   (for example mounted at /data) and set DATA_DIR=/data, otherwise the database
   is wiped on every deploy. Set APP_URL to your public https URL, JWT_SECRET
   to a long random string, NODE_ENV=production, and TZ to your timezone.
4. **Stripe webhook:** add an endpoint at `APP_URL/api/stripe/webhook` for the events
   checkout.session.completed, customer.subscription.updated and
   customer.subscription.deleted. Copy its signing secret into STRIPE_WEBHOOK_SECRET.
5. **Test in Stripe test mode** (card 4242 4242 4242 4242): sign up, add 3 invoices,
   hit the limit, upgrade, send an email to yourself, then cancel from the portal.
6. Swap in live Stripe keys when everything works.

## Not included yet (add before scaling)
- Password reset and email verification
- Terms of service and privacy policy pages (you are emailing people on customers' behalf)
- Unsubscribe handling, and stronger rate limiting
- A backup plan for the SQLite file (or move to Postgres if you grow)
- Automated tests. I could only syntax-check this code here, not run it, so
  expect to fix small issues on first run.
