# Quick Start Guide - Survey-Based Feedback System

## For Administrators

### Creating a New Survey

1. **Login to Admin Panel**
   - Click "Admin" button
   - Enter email: `swagat.kumar@gmail.com`
   - Enter password

2. **Create Survey**
   - In the green "Survey Management" section:
   - Enter **Module Code** (e.g., "CS101")
   - Select **Lecture Date**
   - (Optional) Select **Expiry Date** to auto-disable form
   - Click **"Create New Survey Link"**

3. **Share Survey Link**
   - Survey appears in the list below
   - Click **"Copy Link"** button
   - Share link with students via email, LMS, etc.

### Managing Surveys

**View Active Surveys**
- All active surveys shown in Survey Management section
- Each shows: module code, dates, creation date, link

**Copy Survey Link**
- Click "Copy Link" button
- Link copied to clipboard
- Paste anywhere to share

**Deactivate Survey**
- Click "Deactivate" button on survey
- Confirm the action
- Students can no longer submit to this survey
- Existing submissions remain viewable

### Viewing Feedback

**Admin Panel Table**
- Shows all submissions from all surveys
- Columns: Clarity, Engagement, Pace, Positive, Negative, Date
- Automatic overall rating calculation
- Real-time updates when new feedback submitted

**Download PDF**
- Click "Download PDF" button
- Generates PDF report with all feedback
- Includes ratings breakdown and comments

**Reset All Feedback**
- Click "Reset All Feedback" button (⚠️ USE WITH CAUTION)
- Confirms before deleting
- Permanently removes all feedback data

---

## For Students

### Submitting Feedback

1. **Open Survey Link**
   - Receive link from instructor
   - Example: `https://example.com/?survey=abc123xyz`
   - Open in any browser

2. **Fill Out Form**
   - Rate **Clarity** (1-5 stars)
   - Rate **Engagement** (1-5 stars)
   - Rate **Pace** (1-5 stars)
   - Write **Positive Feedback** (optional)
   - Write **Negative/Improvement Feedback** (optional)

3. **Submit**
   - Click "Submit Feedback"
   - See "Thank you" message
   - Feedback submitted anonymously

### After Submission

**Re-opening Link**
- Shows "Thank you, your feedback has been submitted"
- Cannot submit again (one-time use)
- Works across browser closures

**Privacy**
- Completely anonymous (no email required)
- Admin cannot identify individual students
- Only aggregate data visible

---

## Common Scenarios

### Scenario 1: Weekly Lecture Feedback
**Setup**:
- Create new survey each week
- Module code: "CS101"
- Lecture date: Specific week's date
- Expiry: Next lecture date

**Process**:
1. Monday: Create survey for Week 1 lecture
2. Share link with students
3. Students submit throughout the week
4. Friday: Check feedback in admin panel
5. Next Monday: Create new survey for Week 2

### Scenario 2: Multiple Sections
**Setup**:
- Create separate survey per section
- Module code includes section: "CS101-A", "CS101-B"
- Same lecture date for all

**Process**:
1. Create survey for Section A
2. Copy link, share with Section A students
3. Create survey for Section B
4. Copy link, share with Section B students
5. View feedback separately or combined

### Scenario 3: Guest Lecture
**Setup**:
- Create one-off survey
- Module code: "Guest-AI-Seminar"
- Expiry: Same day (end of event)

**Process**:
1. Create survey before event
2. Share link at end of presentation
3. Students submit during/after event
4. Survey auto-expires at midnight
5. Review feedback next day

---

## Troubleshooting

### Problem: "Survey not found or has expired"
**Cause**: Survey ID invalid, deactivated, or past expiry date
**Solution**: Admin should create new survey or check survey status

### Problem: "Already submitted" but student hasn't
**Cause**: Another student on same device submitted
**Solution**: Each device can only submit once per survey (by design)

### Problem: Cannot create survey
**Cause**: Not logged in as authorized admin
**Solution**: Login with swagat.kumar@gmail.com

### Problem: Survey link not working
**Cause**: Link copied incorrectly or survey deactivated
**Solution**: Re-copy link from admin panel or create new survey

### Problem: Feedback not appearing in admin panel
**Cause**: Firestore rules not deployed or network issue
**Solution**: Check Firebase Console, redeploy rules

---

## Best Practices

### For Administrators

✅ **Create surveys in advance** - Don't wait until lecture ends
✅ **Set expiry dates** - Prevents late submissions, keeps data organized
✅ **Use clear module codes** - Include section/week for clarity
✅ **Download PDFs regularly** - Backup feedback data
✅ **Deactivate old surveys** - Clean up survey list
✅ **Test before sharing** - Open link in incognito to verify

❌ **Don't reuse links** - Create new survey per lecture
❌ **Don't delete surveys immediately** - Wait to collect late feedback
❌ **Don't share admin credentials** - Keep admin access secure

### For Students

✅ **Submit promptly** - While lecture is fresh in mind
✅ **Be specific** - Provide actionable feedback
✅ **Be constructive** - Focus on improvements
✅ **Check expiry** - Submit before deadline

❌ **Don't share links** - (Unless instructor approves)
❌ **Don't submit multiple times** - One submission per survey

---

## Migration from Legacy System

### Old System (Deprecated)
- Global feedback form
- Manual "Make form available" checkbox
- Session-based tracking (unreliable)

### New System (Current)
- Survey-based with unique links
- Automatic availability via active/expired status
- Token-based tracking (reliable)

### Transition Steps

1. **Continue using legacy for existing feedback**
   - Old feedback data still visible in admin panel
   - No data loss

2. **Start creating surveys for new lectures**
   - Use Survey Management section (green box)
   - Share survey links with students

3. **Gradually phase out legacy settings**
   - Legacy section marked with ⚠️ warning
   - Eventually will be removed

4. **Update your workflow**
   - Create survey → Share link (new)
   - Instead of: Enable form → Share general link (old)

---

## FAQ

**Q: Can students edit their submission?**
A: No, submissions are immutable once submitted.

**Q: Can I see who submitted what?**
A: No, all feedback is completely anonymous.

**Q: How many surveys can I create?**
A: Unlimited. Create one per lecture/event as needed.

**Q: What happens to old feedback?**
A: Old feedback remains accessible in admin panel indefinitely.

**Q: Can students submit from mobile?**
A: Yes, fully responsive design works on all devices.

**Q: What if I accidentally deactivate a survey?**
A: Contact developer to reactivate (currently no self-service reactivation).

**Q: Can I change survey details after creation?**
A: Currently no (future enhancement). Create new survey if needed.

**Q: How do I export feedback to Excel?**
A: Use PDF download, then manually transcribe or use OCR.

---

## Support & Feedback

**For technical issues:**
- Check browser console for errors (F12)
- Verify Firestore rules are deployed
- Check Firebase Authentication status

**For feature requests:**
- Document desired functionality
- Provide use case examples
- Submit to development team

**For bug reports:**
- Describe steps to reproduce
- Include browser/device info
- Provide screenshots if applicable

---

**Last Updated**: January 2024  
**Version**: 2.0 (Survey-Based System)  
**System Status**: Production Ready
