# SharePoint 2010 Modal Dialog Wrapper

This script enables an administrator to have users create new rows within a SharePoint (SP) list, but do so in a way without exposing the built-in List Web Part or navigating to the list directly. The benefits of this are that it allows for greater customization, and potential callbacks and editing of a submitted row item after a user has saved the data.

## Installation

If you don't want to insert any server-side code in your SP environment, the easiest way to add content to a given page is to add a Content Editor Web Part (CEWP), and link it to a file that was previously uploaded to a document library. I've found .txt actually works best since SP doesn't cache these by default. This is useful if you are updating files often, and have run into problems with SP's BLOB cache before. Once the CEWP is linked to your source .txt file, uploading and overwriting the file creates changes within the CEWP instantly.

## Usage

This method came about when trying to create automatic and customized row inserts from a application / client-side perspective.

Imagine the following scenario:

A site requests a "quote" submission form, where emails are to be generated automatically from those submissions. Quote requests should also contain file attachments. Using out-of-the-box SharePoint tools, we can create a list with columns for each question we want answered in our quote request, and enable attachments for individual rows. Once a new row is created, a SharePoint Designer Workflow sends an email using the newly create row data in a formatted manner.

However, SharePoint Designer (SPD) does not expose the URLs for row attachments. How can we access this data *without* adding any event listeners on the server side (aka, accomplish this all from the client side)? 

Enter our **modal wrapper**. Now, we can create an experience that will go something like this:

1. A user clicks the "Request Quote" button.
2. Using JSOM, a new row in our SP List is created. A boolean column within our list titled **SendEmail** is given an initial value of `FALSE`. This column is also hidden from our form view (by managing the content types).
3. Upon row creation, an SPD Workflow is activated. That workflow is very simple: `IF CurrentItem:SendEmail equals TRUE, THEN Send-Email`. So in this case, our newly created row has a SendEmail value of `FALSE`, so no email is sent, and the workflow has finished.
4. After the row item is created, using the SP.UI built-in methods, we open a Modal Dialog window and load the EditForm.aspx page, and pass in the ID of the newly created row. All values of our previously created row were initially blank, so even though the user is technically editing an existing row, to them, it appears that they are creating a new row.
5. The user enters the information, and saves the data. Again, the **SendEmail** col is still false (remember this is hidden from our form view), so our SPD Workflow does not send an email.
6. After saving the data, we perform a callback, and query the newly updated row using jQuery's ajax and retrieve the URLs for any attached files. We then take those URLs, format them to HTML links,and update the our row and insert those formatted URLs into another column (say AttachmentUrls). We now have everything saved in our list that we'd want to display when we send our email, so we additionally update the **SendEmail** col to be `TRUE` at the same time.
7. Our SPD Workflow kicks off, sees that **SendEmail** is `true`, and sends our email, pulling data from cols within our SP List and formatting the email accordingly.
8. Note, if the user would have canceled out of the modal window instead of saving, we'd have a blank row within our SP List floating around. Thus, if the user cancels, we delete the row automatically.

In this scenario, after several uses, you will have an SP List which contains all of your quote requests, and your team which the quote requests are intended for is receiving automated emails containing all of their relative information. All this while not exposing the raw SP List interface to the user, allowing for better brand styling and user experience. We also get the benefits of still using out-of-the-box form elements which let us take advantage of SP's WYSIWYG editor, and their File Attachment support on the per-row basis.

## Cons

The primary con of this method is rows *must* be submitted through this custom application. If you add elements to the raw manually through the List interface, our custom callbacks will not be fired, and our SPD Workflows may never fully process. However, if a server-side solution is not viable, this client-side solution certainly gets the job done.
