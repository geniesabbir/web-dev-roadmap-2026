# Job Preparation

**Duration:** 2-3 weeks

## Learning Objectives

By the end of this section, you will:
- Have a polished portfolio
- Write effective resumes
- Ace technical interviews
- Negotiate job offers

---

## Week 1: Portfolio & Resume

### Portfolio Requirements

**Must-Have Projects**
1. **Full-Stack Application** - Shows end-to-end skills
2. **Complex Frontend** - Demonstrates React mastery
3. **API/Backend** - Shows server-side capabilities
4. **Open Source Contribution** - Shows collaboration

**Portfolio Website Checklist**
- [ ] Clean, professional design
- [ ] Mobile responsive
- [ ] Fast loading
- [ ] Project showcases with:
  - Screenshots/demos
  - Tech stack used
  - Your role/contributions
  - Live demo links
  - GitHub links
- [ ] About section
- [ ] Contact information
- [ ] Resume download

### Resume Guidelines

**Format**
- 1 page (2 max for senior)
- Clean, readable font
- Consistent formatting
- PDF format

**Sections**
1. Contact Info (name, email, LinkedIn, GitHub, portfolio)
2. Summary (2-3 sentences)
3. Skills (be specific)
4. Experience (most recent first)
5. Projects (if limited experience)
6. Education

**Writing Tips**
```
BAD: "Worked on the frontend team"
GOOD: "Built 5 customer-facing features using React, reducing bounce rate by 20%"

BAD: "Responsible for database management"
GOOD: "Optimized PostgreSQL queries, improving API response time from 2s to 200ms"

BAD: "Used many technologies"
GOOD: "Stack: React, TypeScript, Node.js, PostgreSQL, Docker, AWS"
```

**Action Verbs**
Built, Developed, Implemented, Designed, Optimized, Led, Reduced, Improved, Automated, Deployed, Architected

---

## Week 2: Technical Interviews

### Coding Interview Patterns

**Array/String Problems**
```typescript
// Two pointers
function isPalindrome(s: string): boolean {
  let left = 0, right = s.length - 1
  while (left < right) {
    if (s[left] !== s[right]) return false
    left++
    right--
  }
  return true
}

// Sliding window
function maxSubarraySum(arr: number[], k: number): number {
  let maxSum = 0, windowSum = 0
  for (let i = 0; i < k; i++) windowSum += arr[i]
  maxSum = windowSum
  for (let i = k; i < arr.length; i++) {
    windowSum = windowSum - arr[i - k] + arr[i]
    maxSum = Math.max(maxSum, windowSum)
  }
  return maxSum
}

// Hash map for frequency
function twoSum(nums: number[], target: number): number[] {
  const map = new Map<number, number>()
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i]
    if (map.has(complement)) {
      return [map.get(complement)!, i]
    }
    map.set(nums[i], i)
  }
  return []
}
```

**Must-Know Concepts**
- Big O notation
- Arrays and strings
- Hash tables
- Stacks and queues
- Trees and graphs (basics)
- Recursion

### System Design Interview

**Template**
1. **Clarify** (2-3 min)
   - What's the scale?
   - What features?
   - What constraints?

2. **High-Level Design** (5-10 min)
   - Draw main components
   - Show data flow

3. **Deep Dive** (15-20 min)
   - Database design
   - API design
   - Scaling strategies

4. **Wrap Up** (5 min)
   - Trade-offs
   - Future improvements

**Common Questions**
- Design a URL shortener
- Design Twitter/Instagram feed
- Design a chat application
- Design an e-commerce site
- Design a rate limiter

### Behavioral Interview (STAR Method)

**Situation** → **Task** → **Action** → **Result**

**Common Questions**
1. Tell me about a challenging project
2. Describe a time you disagreed with a teammate
3. How do you handle tight deadlines?
4. Tell me about a bug you debugged
5. Describe your biggest failure

**Example Answer**
```
Q: Tell me about a challenging project

Situation: Our team needed to migrate a legacy PHP application to React.

Task: I was responsible for the frontend architecture and leading
the migration of 50+ components.

Action: I created a migration plan, set up the new React architecture
with TypeScript, implemented a component library, and trained team
members on React best practices.

Result: Successfully migrated in 3 months, improved page load time
by 60%, and reduced bug reports by 40%.
```

---

## Week 3: Job Search

### Application Strategy

**Where to Apply**
- LinkedIn Jobs
- Indeed
- Company career pages
- AngelList (startups)
- Wellfound
- Remote-specific: RemoteOK, WeWorkRemotely

**Application Tips**
- Customize resume for each role
- Write targeted cover letters
- Apply to 5-10 quality jobs vs 50 random ones
- Follow up after 1 week
- Network on LinkedIn

### Interview Process

**Typical Flow**
1. Resume screen
2. Recruiter call (15-30 min)
3. Technical screen (1 hour)
4. Technical interviews (2-4 hours)
5. System design (1 hour)
6. Behavioral (1 hour)
7. Team fit / culture (30-60 min)
8. Offer

### Salary Negotiation

**Research**
- Glassdoor, Levels.fyi, Blind
- Know market rate for your level and location

**Tips**
1. Don't give a number first
2. If pushed: "Based on my research, roles like this pay $X-Y"
3. Negotiate total compensation (base, bonus, equity, benefits)
4. Get offers in writing
5. It's okay to ask for time to decide

**Phrases**
- "I'm excited about this role. Based on my research and experience, I was expecting something closer to $X"
- "Is there flexibility in the base salary?"
- "I have competing offers I'm considering. Is there room to improve this offer?"

---

## Interview Checklist

### Before the Interview
- [ ] Research the company
- [ ] Review job description
- [ ] Prepare questions to ask
- [ ] Test your setup (camera, mic, IDE)
- [ ] Have water nearby

### During the Interview
- [ ] Think out loud
- [ ] Ask clarifying questions
- [ ] Start with brute force, then optimize
- [ ] Test your solution
- [ ] Be honest about what you don't know

### Questions to Ask
- What does a typical day look like?
- How is the team structured?
- What's the tech stack?
- How do you handle code reviews?
- What does success look like in this role?
- What are the biggest challenges?

---

## Resources

### Coding Practice
- [LeetCode](https://leetcode.com/) - Start with Easy, move to Medium
- [NeetCode](https://neetcode.io/) - Curated problem sets
- [Exercism](https://exercism.org/)

### System Design
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [ByteByteGo](https://bytebytego.com/)

### Mock Interviews
- [Pramp](https://www.pramp.com/) - Free peer practice
- [Interviewing.io](https://interviewing.io/)

---

## Final Checklist

- [ ] Portfolio website live
- [ ] 3+ quality projects showcased
- [ ] Resume polished and reviewed
- [ ] LinkedIn profile updated
- [ ] GitHub profile cleaned up
- [ ] Practiced 50+ coding problems
- [ ] Prepared system design answers
- [ ] Prepared behavioral answers
- [ ] Ready to apply!

---

**Congratulations!** 🎉

You've completed the Full Stack Developer Roadmap. You now have:
- Strong fundamentals in frontend and backend
- Experience with modern frameworks and tools
- Understanding of DevOps and deployment
- Knowledge of advanced topics
- A portfolio to showcase your skills

**Next Steps:**
1. Build your capstone project
2. Apply for jobs
3. Never stop learning

Good luck on your journey!
