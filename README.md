# feedback-form
To implement a dynamic question flow where the user is shown one question at a time and moves to the next question upon submission, you'll need to adjust both the backend logic and frontend to handle sequential question delivery. This approach involves keeping track of the current question and loading the next question without reloading the page, using AJAX for data fetching and submission.

### Backend Adjustments

First, let's set up an endpoint to fetch the next question. This example assumes that questions are ordered or have an implicit order (e.g., by their `id` or a specific `order` field).

#### models.py
No change is needed here, but ensure your `Question` model has a way to determine the order if not using the primary key `id`.

#### views.py
Add a view to fetch the next question. This example uses a simple mechanism assuming questions have an incremental ID or a predefined order.

```python
from django.http import JsonResponse
from .models import Question

def get_next_question(request, current_question_id=None):
    if current_question_id is None:
        # Get the first question
        next_question = Question.objects.order_by('id').first()
    else:
        # Get the next question based on the current question ID
        next_question = Question.objects.filter(id__gt=current_question_id).order_by('id').first()
   
    if next_question:
        return JsonResponse({
            "success": True,
            "question_id": next_question.id,
            "question_text": next_question.text,
        })
    else:
        return JsonResponse({"success": False, "message": "No more questions."})
```

### Frontend Adjustments

Modify your HTML and JavaScript to handle one question at a time and request the next question upon submission.

#### HTML Template
Adjust the form to have a placeholder for the question and an input field for the answer. Initially, you might not display any question until fetched by JavaScript.

```html
<form id="feedbackForm" method="post">
    <div id="questionContainer">
        <!-- Question will be loaded here -->
    </div>
    <button type="submit" style="display:none;">Submit Answer</button>
</form>
```

#### JavaScript for Dynamic Question Flow
Update your JavaScript to fetch and display questions one at a time.

```html
<script>
$(document).ready(function() {
    let currentQuestionId = null;

    function fetchNextQuestion() {
        $.ajax({
            url: `/get-next-question/${currentQuestionId || ''}`, // Adjust URL as needed
            success: function(response) {
                if (response.success) {
                    $('#questionContainer').html(`
                        <label>${response.question_text}</label>
                        <input type="text" name="answer" data-question-id="${response.question_id}">
                    `);
                    $('button[type="submit"]').show();
                    currentQuestionId = response.question_id;
                } else {
                    $('#questionContainer').html(`<p>${response.message}</p>`);
                    $('button[type="submit"]').hide();
                }
            }
        });
    }

    fetchNextQuestion(); // Load the first question on page load

    $('#feedbackForm').submit(function(e) {
        e.preventDefault();
        const answer = $(this).find('input[name="answer"]').val();
        const questionId = $(this).find('input[name="answer"]').data('question-id');

        // Logic to submit the answer goes here
        // On successful submission, fetch the next question
        console.log(`Submitting answer "${answer}" for question ID ${questionId}`);
        // After submitting the answer, fetch the next question
        fetchNextQuestion();
    });
});
</script>
```

### Considerations and Next Steps
- **Submitting Answers:** The example shows how to handle the question flow, but you'll need to integrate the logic to submit each answer to your backend and potentially link it to a feedback session. You might adjust the `submit_feedback` view or create a new one specifically for handling individual answers.
- **End of Questions:** The script checks for more questions and hides the submit button when there are no more. You may want to handle this more gracefully, perhaps by showing a completion message or redirecting the user.
- **Session Management:** Consider how you'll track which feedback session these questions and answers belong to. You might start a feedback session when the first question is fetched and pass a session identifier along with each answer.

This setup provides a dynamic, single-question-at-a-time feedback experience, enhancing user engagement and making the feedback process more interactive.
