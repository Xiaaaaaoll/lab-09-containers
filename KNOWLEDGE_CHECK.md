# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

Anyone who pulls the public image could extract my AWS credentials from its layers and use them to access AWS resources with my permissions. The credentials would also expire when my Learner Lab session ends, so the image would still contain the old credentials and the analysis would fail.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

Since I left out the -e options, get_atlas_settings() didn’t find MONGODB_ATLAS_URL and fell back to AWS Systems Manager Parameter Store. Boto3 read my AWS credentials from the read-only mount at /home/app/.aws and used them to request the decrypted mongodb_credentials parameter in us-east-1. The function parsed the returned JSON to get the Atlas URL, username, and password.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Reusing 0.1.0 means someone with a cached copy could use the old image while someone pulling the same tag could get the new one, making it harder to reproduce the exact run. I should have kept 0.1.0 pointing to the original image and tagged the updated image as 0.1.1.

The image ID before the rebuild was c1c0649ed9c0, and the image ID after the rebuild was 0f8b3c88e037.

