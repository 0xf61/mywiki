---
title: Resources
draft: false
---

I use [Firecrawl](https://www.firecrawl.dev/) and simple AI tools to build my own online pocket alternative.
I save webpages that I really enjoy reading or want to mention in my posts.

This part of the wiki is automatically generated, so it may not be in perfect condition and could contain broken links or styling issues.
When I review and update these resources, they will be moved to the [[coolThingsOnline]] folder.
If you are the owner of any content listed here and would like it removed, please contact me and I will take it down.

If you want to submit an URL you can use here:

<form id="urlForm" onsubmit="submitUrl(event)">
  <label for="url">URL:</label>
  <input type="url" id="url" name="url" placeholder="example.com" required>
  <button type="submit">Submit</button>
</form>

<script>
function submitUrl(event) {
  event.preventDefault();

  const url = document.getElementById('url').value;

  // Using GET request to the n8n webhook with URL as a query parameter
  fetch('https://n8n.de2.in/webhook/submit?url=' + encodeURIComponent(url), {
    method: 'GET'
  })
  .then(response => {
    console.log('Success:', response);
    alert('URL submitted successfully!');
    document.getElementById('url').value = '';
  })
  .catch((error) => {
    console.error('Error:', error);
    alert('Error submitting URL. Please try again.');
  });
}
</script>
