# Exercise Feedback — Sofia M.

## 1. What parts of the exercise were clear and helpful?

- The **phased approach** (Phase 1 through 4) was really helpful. It stopped me from jumping straight into endpoints, which I probably would have done otherwise. Having a clear order to follow made the whole thing less overwhelming.
- The **worked example** for `POST /api/v1/members` was great. I kept going back to it to check if my level of detail was right. Without it, I would have had no idea how detailed to get.
- The **templates** for each phase were useful. I basically copied them and filled them in. Having that structure saved me a lot of time figuring out how to format things.
- The **business rules** being listed out as numbered rules was really clear. It made it easy to check off which ones I'd covered.
- The **ER diagram** in ASCII was simple but it helped me visualize the relationships quickly.

## 2. What parts were confusing or overwhelming?

- The **qualification matching** (Rule 4) was confusing. The rule says the instructor needs a "matching qualification" but it's not clear what "matching" means. Does the class title have to exactly match one of the qualifications? What if the class is called "Morning Yoga" — does that match "Yoga"? I had to make up my own rule for this and I'm not sure I got it right.
- The **Design Decisions** section lists a bunch of trade-offs but doesn't tell me which to pick. I get that that's the point, but it felt like a lot of open questions all at once. I just picked the simplest option each time because I didn't want to overthink it.
- The **overlap checking** for rooms and instructors took me a while to think through. I had to figure out the math for when two time ranges overlap. It's not super hard but it was a detail I hadn't thought about before.
- The difference between `400`, `404`, and `409` was sometimes unclear. Like when a member is inactive and tries to book — is that a `400` (bad request) or a `409` (conflict)? I went with `409` but I'm not fully sure.

## 3. Where did you get stuck and why?

- I got stuck on **how to handle the qualification matching**. The exercise doesn't define what qualifications look like or how they map to class titles. I spent a while going back and forth and eventually just decided on substring matching.
- I got a bit stuck on **what to do with PATCH on classes** when it comes to capacity reduction. The exercise mentions it in the design decisions section but it took a while to figure out what status code to use and how to phrase the error.
- I wasn't sure if I needed a separate endpoint to list a **member's bookings**. The exercise says bookings are under classes, but it seems like a frontend would want to show "my bookings" for a member. I decided not to add it to keep things simpler but I'm not sure if that was the right call.

## 4. What would you change about the exercise?

- **Clarify the qualification matching rule.** Either give us a fixed list of valid qualifications, or explain more clearly how the matching between class title and instructor qualification is supposed to work.
- **Maybe provide a second worked example** for a more complex endpoint like creating a class or creating a booking. The member creation example is helpful but it's one of the simpler endpoints. I would have liked to see how to handle an endpoint with multiple business rules.
- The **"Design Decisions to Think About" section** could have a hint about which option is simpler or more common. I know the point is to make us think, but when you're new to API design, having some guidance on what's the "normal" approach would be nice.
- **The time estimate might be a bit tight** for students who haven't done this before. Phase 4 took me longer than expected because I kept second-guessing my error responses and status codes.

## 5. How long would this realistically take?

I'd estimate about **6-7 hours** of focused work. Phase 1 and 2 went pretty fast (about an hour total). Phase 3 was quick too (maybe 20 minutes). But Phase 4 took the bulk of the time — probably 4-5 hours — because each endpoint requires you to think about multiple error cases and business rules. The booking and class creation endpoints especially took a while because they have so many rules to enforce.

If I was doing this in one sitting I'd probably need breaks, so realistically it might be spread over two sessions.

## 6. Difficulty Rating: 6/10

It's challenging but doable. The exercise is well-structured enough that you always know what to do next, which helps a lot. The hard part isn't any single concept — it's the sheer amount of detail you need to think through for every endpoint. You have to keep track of all the business rules and make sure each one is reflected somewhere in your spec.

The advanced/optional topics (pagination, concurrency, etc.) look significantly harder. I didn't attempt any of them. If those were required, this would be an 8/10 easily.

What made it manageable: the templates, the worked example, and the phased approach. Without those, I think it would have been much harder to know where to start or when you're "done enough."
