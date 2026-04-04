# Exercise Feedback - Marco

## 1. What parts of the exercise were clear and helpful?

- The **phased approach** (Phase 1 through 4) was really helpful. Without that I would have just started writing random endpoints and gotten lost. Having the order spelled out made it feel less overwhelming.
- The **templates** for each phase were great. I basically just copied them and filled them in. Without those I would have stared at a blank page for an hour.
- The **worked example** (POST /members) was probably the most useful part. I kept going back to it to check "am I doing this right?" I wish there were more examples like that, maybe for a more complex endpoint.
- The **business rules list** being numbered was nice because I could reference them in my endpoint specs (like "enforces Rule 7").
- The **ER diagram** at the top was simple and easy to understand. Helped me see the big picture before diving in.

## 2. What parts were confusing or overwhelming?

- **Rule 4 (qualification matching)** really confused me. The class has a "title" like "Morning Yoga" and the instructor has qualifications like "Yoga". But how do you match those? Should the class have a separate "type" or "category" field? The exercise doesn't say. I wasted a lot of time going back and forth on this and I'm still not happy with my solution. I kind of just... hand-waved it.
- The **Design Decisions section** lists a bunch of questions (bookings as a resource, qualifications modeling, cancellation vs deletion, etc.) but doesn't give any guidance on which choice is better for beginners. I get that there's "no wrong answer" but when you're already confused, having 6 open questions with no hints is stressful. Maybe suggest a default choice and explain why someone might deviate?
- **How overlap checking works** (Rules 5 and 6) -- I said the API should check for overlaps but I'm honestly not sure what the error response should look like or how to describe the overlap detection logic in enough detail for an LLM to implement it. Like, do I need to specify the SQL query or algorithm?
- The **"May Include (Advanced)" section** made me feel like my solution is incomplete even though it says it's optional. I didn't do any of it.

## 3. Where did you get stuck and why?

- **Qualification matching** (see above). This was my biggest blocker. I spent probably 30 minutes going in circles.
- **Booking as nested resource vs top-level.** I went with nested but then realized there's no clean way to get "all bookings for member X" without adding another endpoint. I felt like whatever I chose was wrong.
- **PUT vs PATCH.** I know they're different but I always forget exactly how. I just used PUT everywhere because it felt safer, but I have a feeling that's not ideal for things like changing just the membership status.
- **Phase 4 took forever.** Even with the "abbreviated" template option, writing out every endpoint with all its error cases was really tedious. I started strong but by the end I was definitely rushing and my later endpoints are less detailed.
- **How detailed is detailed enough?** The exercise says "an LLM should be able to implement it without asking questions" but I don't know what level of detail an LLM needs. Like, do I need to specify what happens to the response when you add a booking -- does the class endpoint show a `currentBookings` count? I just didn't think about stuff like that.

## 4. What would you change about the exercise to make it better for students like me?

- **Add a second worked example** for a more complex endpoint, like creating a class (with the qualification and overlap checks). The member creation example is great but it's the simplest case.
- **Give a hint about qualification modeling.** Even just saying "consider adding a `type` field to classes" would save a lot of confusion.
- **Reduce the number of design decisions** that are left open, or provide a "recommended default" for beginners. Advanced students can deviate.
- **Add a midpoint checkpoint.** Something like "after Phase 2, your document should look roughly like this" so I can check if I'm on track before investing 3 more hours.
- **Maybe provide the endpoint overview table already filled in** (Phase 3) and let students focus on Phase 4? Listing the endpoints wasn't the hard part -- detailing them was.
- **Clarify the overlap-checking rules.** A concrete example would help: "Class A is in Room 1 from 10:00-11:00. Class B tries to book Room 1 from 10:30-11:30. This is an overlap because..." I know this sounds obvious but when you're trying to write it as a spec, the edge cases get confusing fast (what about exact boundary touching? Is 11:00-12:00 an overlap with 10:00-11:00?).

## 5. How long would this realistically take me?

Honestly? Probably **6-7 hours** spread over 2-3 sessions. The first 2 phases I could do in about an hour. Phase 3 maybe another 30-40 minutes. But Phase 4 is where it gets really time-consuming, especially if you're trying to be thorough about error cases and examples. I'd probably take a break after doing the member and instructor endpoints and come back later for classes and bookings.

If you factor in the time I spent re-reading the instructions, second-guessing my design decisions, and going back to fix inconsistencies... yeah, 6-7 hours minimum. Faster students could probably do it in 4-5.

## 6. Difficulty Rating: 7/10

**Why:** The exercise itself is well-structured and the templates help a lot. But there's a LOT to think about. The business rules add real complexity, especially the overlap checks and qualification matching. The hardest part isn't any single endpoint -- it's keeping everything consistent across 18 endpoints while also making design decisions you've never made before. For someone like me who hasn't designed an API before, it's challenging but doable. It would be an 8 or 9 without the templates and worked example.
