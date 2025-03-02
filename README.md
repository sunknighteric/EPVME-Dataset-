# EPVME-Dataset
The five zip packages are all the malicious samples, totaling 49,136 EML files. This dataset is for research purpose.

## Dataset Description
Here are reference links of other open-source datasets we used in our work.

Enron Email Dataset "http://www.cs.cmu.edu/~./enron/"

Nazario phishingcorpus "https://monkey.org/~jose/phishing/"

Spam Assassin open-source corpus "https://spamassassin.apache.org/old/publiccorpus/"

TREC 2007 "https://plg.uwaterloo.ca/~gvcormac/treccorpus07/"

Wooyun Email XSS Dataset "https://github.com/WhiteRabbitc/Wooyun-Email-XSS-Dataset/tree/master/malious-sample"

Email Dataset Sample Amount:
| Source                         | Malicious | Benign  | Total   |
|--------------------------------|-----------|---------|---------|
| Enron email corpus             | 0         | 517,401 | 517,401 | 
| Nazario phishing corpus        | 9,510     | 0       | 9,510   | 
| Trec07                         | 50,199    | 25,220  | 75,419  | 
| Spam Assassin                  | 2,399     | 6,952   | 9,351   | 
| Wooyun XSS Dataset             | 168       | 0       | 168     |  
| EPVME Dataset                  | 49,136    | 0       | 49,136  |   
| Total                          | 111,412   | 549,573 | 660,985 |  


## How we make the malicious email samples
We select and summarize six new types of malicious email attack scenarios that exploit vulnerabilities in email security authentication mechanisms (SPF and DMARC) and email UI rendering, and construct the EPVME dataset. It contains 37,283 malicious emails, and all email samples have email header and body information.

In the malicious samples we created, except for some specific fields where we inject the payload, other content, including *From*, *To*, *Subject*, and body text, is randomly selected from the existing samples. This is because we believe that the detection model should be able to identify malicious emails that use these attacks without being influenced by this irrelevant content. This allows the machine learning model to focus on static features related to the payload and thus achieve the identification of these new types of malicious emails. 

### Inconsistency between Mail From and From header attacks
For the Inconsistency between Mail From and From header attacks, the value of the *Mail From* field is randomly selected from the sender email addresses of malicious email samples, and the value of the *From* field is generated from the sender email addresses of all benign emails. For the Empty Mail From attack, we set the *Mail From* field to "<>". The value of the *From* field is generated from the sender email addresses of all benign emails, and the body is randomly selected from malicious samples.

<img width="326" alt="1679556723667" src="https://user-images.githubusercontent.com/32115816/227133907-d0a440ad-92fb-43b4-9729-483db18e6f29.png">

### Multiple From header attacks
We choose email addresses from both benign and malicious samples and set them as two values of *From*. Based on these four scenarios, we create malicious email samples equally.

<img width="319" alt="1679556835716" src="https://user-images.githubusercontent.com/32115816/227134285-de74317e-2f21-4b4d-bfba-9391f3ebae42.png">

### From Header Injection
We use unicode characters, utf-8 encoding, base64 encoding, and other injection methods to construct the *From* field, and the body part is selected from the malicious sample to construct an email sample with a legitimate recipient but malicious body content.

<img width="257" alt="1679556882412" src="https://user-images.githubusercontent.com/32115816/227134428-0a455d02-a28e-4976-a2ac-13d31e734b68.png">

### Subdomain Attacks
As with subdomain attacks, we use subdomains of legitimate sender domains (test.domain.com or security.domain.com) to construct the header information along with the body content of the malicious emails in the previously collected dataset to build the complete malicious email samples.

### XSS
We divide the payloads into two categories. One category is malicious code constructed from a single HTML statement. Since the former works in both the header and body of the email, we insert each specific statement equally into the *From*, *To*, *Subject*, and *Body* fields of the email body. For the three header fields, we add the payload above the sample "Subject: XSS test mail", "From: <test@attack.com>", "To: <victim@victim.com>". The other fields and content are filled in randomly from existing benign email samples or by using test samples. The other type is longer, the payload is more complex, and more techniques are used, such as encoding, which is only applicable to be placed in the body part of the email. We randomly select the content from the good emails to fill the headers of the emails and insert the payload in the body, and some emails will also add some body content from the benign emails after the payload.

### CMS Attacks

Cryptographic Message Syntax (CMS) is a common standard for signing and encrypting email that is used by S/MIME. The CMS and S/MIME standards define two types of signed messages: opaque signatures and split signatures. The main difference between the two types is that the former encrypts the text content to be signed along with the eContent, while the latter separates the content to be signed from the signed encrypted message when the *Content-Type* is set to *multipart/signed* and there is no eContent. If an attacker obtains a signed email, the attacker can construct arbitrary content in the body by removing the original content and adding an illegitimate eContent field to disguise the authentication of the email recipient. Due to a design flaw in S/MIME, an email may have multiple signatures or no signatures at all, so an attacker can use this vulnerability to construct spoofed emails. The CMS attack type of malicious email sample construction mainly modifies the signature encryption part of legitimately signed emails by adding more signer information, or simply removing all signature information, etc., which will eventually lead to abnormal base64 character lengths of signatures in these malicious email samples that use digital signatures.

### MIME Attacks

If an attacker already has at least one email and a corresponding valid signature from the impersonated entity based on MIME syntax rules, the attacker can manipulate the content of the email to deceive the recipient in a number of ways, including (1) Adding fake content and a large number of blank lines before the signed email so that the recipient notices only the fake content in the email and knows that the e-mail has been signed. (2) Using HTML/CSS tags to hide the legitimate body information of the signed email. (3) Using *cid.references* to link the unsigned malicious text to the signature section. (4) Marking the signed section as an attachment so that it can be hidden.

Next two figures are two typical examples of MIME attacks. The former uses a large number of line breaks to make the recipient see only the malicious text and thus ignore the benign signed email content. The latter uses HTML/CSS to hide the signed part so that the recipient cannot see it in the email client.

<img width="334" alt="1680070415967" src="https://user-images.githubusercontent.com/32115816/228442531-0626bce4-4873-4ef3-820a-d8b4c70283ce.png">

<img width="368" alt="1680070431972" src="https://user-images.githubusercontent.com/32115816/228442586-d3e3b9a7-818d-4992-a82a-8612373a4415.png">

### ID Attacks

This type of attack mainly exploits the difference between the information in the signature and the identity of the sender. However, many email receiving clients allow users to check the details of the signature, which may show signs of tampering, thus reducing the success rate of this type of attack. The main forms of this type of attack include: (1) The receiving server may not be able to verify that the sender and signer are the same, so the attacker can sign his own message and impersonate a legitimate recipient. (2) Based on (1), the sender field is constructed through a possible vulnerability in the UI display on the receiving end, so that the recipient can see the legitimate sender, but the actual sender is the attacker. (3) The SMTP protocol has vulnerabilities allowing multiple sender fields and possible inconsistencies in the *Mail From* and *From* fields. When the recipient of the mail checks the signature, it may traverse the sender, and after the attacker signs with his own information, the recipient's UI may display the legitimate sender. 

To summarize these three types of attacks, the core lies in the vulnerability of the SMTP protocol, which allows multiple *From* fields. By modifying the header information in the form of Multiple From Header, it is possible to make the sender information seen by the recipient and the person who actually signs the email inconsistent.
Next figure is an example of From Header Confusion. There are two *From* fields in the email, and some email clients display the sender information in the first *From* field. Because this signature uses the attacker's information, the email is marked as having a legitimate digital signature.

<img width="397" alt="1680070532386" src="https://user-images.githubusercontent.com/32115816/228442881-c43c4da1-17e3-48e4-8a64-281247ce080b.png">

## Appendix

Here are the full list of the static features that we extract from all email samples to realize the validation of EPVME dataset.

### Header Features

| Feature Name                         | Description | Domain |
|--------------------------------|-----------|-----------|
| Subject IsReply             | If the email is a reply to a previous email from the sender.           |{0,1} |
|Subject IsForwarded|If the email is forwarded from another account to the recipient.|{0,1} |
|Subject NumWords |Number of words in the subject line of the email. |N|
|Subject NumCharacters|Number of characters in the email's subject line.|N|
|Subject Richness|Richness of the subject line.|[0,1]|
|Subject Verify |If subject field has keyword 'verify'. |{0,1}|
|Subject Debit| If subject field has keyword 'debit'. |{0,1}|
|Subject Bank|If subject field has keyword 'bank'.|{0,1}|
|Subject Account| If subject field has keyword 'account'. |{0,1}|
|Subject Approve|If subject field has keyword 'approve' or 'approval'. |{0,1}|
|Subject Buy |If subject field has keyword 'buy'.  |{0,1}|
|Subject Earn|If subject field has keyword 'earn'. |{0,1}|
|Subject Family  |If subject field has keyword 'family'. |{0,1}|
|Subject Guarantee|If subject field has keyword 'guarantee'. |{0,1}|
|Subject Hello|If subject field has keyword 'hello'. |{0,1}|
| Subject Money|If subject field has keyword 'money'. |{0,1}|
|Subject Only|If subject field has keyword 'only'.   |{0,1}|
|Subject Own|If subject field has keyword 'own'.|{0,1}|
|Subject ?Or! |If subject field has keyword '?' or '!'.|{0,1}|
|Subject Save|If subject field has keyword 'save' or 'saving'.|{0,1}|
|Subject Statement |If subject field has keyword 'statement'.|{0,1}|
|Subject Gapped| If subject field is gapped by underlines or spaces. |{0,1}|
|Subject HasUsername| If user nickname in from field shows in subject.|{0,1}|
|Subject Coded|If subject field contains non-ASCII characters. |{0,1}|
|Subject HasHTML|If subject field contains HTML tags.  |{0,1}|
|Received TotalNum|Number of “Received: . . . ” fields.  |N|
|Send NoWords |Number of words in the send field.          |N|
|Send NoCharacters| Number of characters in the sender field.|N|
|Send DiffSenderReplyTo|If there is a difference between the sender's domain and the reply-to domain. |{0,1}|
|Send NonModalSenderDomain |If there is a difference between the sender's domain and the reply-to domain.|{0,1}|
| From 2Addr|If from field contains more than one email address. |{0,1}|
|From Coded |If from field is coded with non-ASCII char set. |{0,1}|
| From SpecialWord|If from field contains keyword 'free', 'no-reply', and 'offer'.|{0,1}|
|From Mixed| If from field contains numbers mixed in with letters. |{0,1}|
|From Lower|If from field does not have lower case letter. |{0,1}|
|From NoAddr|Number of valid addresses in from field. |N|
|From NoUser|Number of users in from field.  |N|
|From HasHTML|If from field contains HTML tags.  |{0,1}|
|From Subdomain|If from field contains subdomain email address.  |{0,1}|
|InReplyTo|If in reply to field exists. |{0,1}|
|MailFrom Bounce|If mailfrom field has keyword 'bounce'. |{0,1}|
|MsgID At|If message id field does not have '@'.|{0,1}|
|MsgID Host|If message id field does not have a valid domain. |{0,1}|
|ReplyTo Mixed|If reply to field exists and contains numbers mixed with letters. |{0,1}|
|ReplyTo Addr| If reply to field exists but no valid address.|{0,1}|
|To Missing|If to field is missing.|{0,1}|
|To Addr|If the to field does not exist or has no valid address.  |{0,1}|
|To Sorted|If more than 2 recipients in to field and in alphabetical order. |{0,1}|
|To HasHTML|If to field contains HTML tags.  |{0,1}|
|Webmail True |If is webmail field is true (email was sent from a web interface).  |{0,1}|
|DEP Mailfrom From|Similarity between mailfrom field address and field from address.|[0,1]|
|DEP Mailfrom Helo | Similarity between mailfrom field and helo field.|[0,1]|
|DEP Mailfrom ReplyTo|Similarity between mailfrom field and field reply to (if exists).|[0,1]|
| DEP MsgID Helo |Similarity between message id field and field helo.|[0,1]|
|DEP In Future|If the sending timestamp is in the future of receiving time stamp. |{0,1}|

Notes:

(1)Subject Richness = Subject NumWords / Subject NumCharacters.

(2)The DEP features are calculated using the Jaccard similarity coefficient.

### Body Features

| Feature Name                         | Description | Domain |
|--------------------------------|-----------|-----------|
| Content Type Total Num              | Number of “Content-Type” fields.          |N |
|Content Type Unique Num |Number of unique “Content-Type” fields. |N|
|Content Type Multipart Mixed Num |Number of “Content-Type: multipart/mixed” fields.      |N|
|Content Transfer Encoding Total Num |Number of “Content-Transfer-Encoding” fields.       |N|
|Content Transfer Encoding Unique Num | Number of unique “Content-Transfer-Encoding” fields . |N|
|Content Transfer Encoding 64bit Num|Number of “Content-Transfer-Encoding: 64bit” fields. |N|
|Content Transfer Encoding 7bit Num |Number of “Content-Transfer-Encoding: 7bit” fields.|N|
|Content Transfer Encoding 8bit Num|Number of “Content-Transfer-Encoding: 8bit” fields.|N|
|Content Transfer Encoding Binary Num|Number of “Content-Transfer-Encoding: binary” fields. |N|
|Content Disposition Total Num  |Number of “Content-Disposition” fields.   |N|
|Content Disposition Unique Num|Number of unique “Content-Disposition” fields.|N|
|Content Type Signed Signature Num|Number of digital signature used.|N|
|Body HasPhonenumber  |If body field has phonenumbers.|{0,1}|
|Body NumUniqueWords   |Number of words occurred in body field.  |N|
|Body NumFunctionWords|Number of occurrences of function words in the email body. |N|
|Body Numlines|Number of lines in body field. |N|
|Body NumOfURLs  |Number of URLs in body field. |N|
|Body NumOfDomains|Number of domains in body field.  |N|
|Body NumWords|Number of total words occurred in body field.  |N|
|Body NumCharacters |Number of characters occuring in the email body.  |N|
|Body Richness |Ratio of the number of words to the number of characters in the document. |[0,1] |
|Body Html|If HTML exists in the body part.|{0,1}|
|Body Form|If HTML form exists in the body part. |{0,1}|
|Url NoIpAddresses|Number of links that contain IP addresses.|N|
|Url IpAddress|Whether existing IP addresses or a qualified domain name in body part.|{0,1}|
|Url AtSymbol|If links contain an '@' symbol. |{0,1}|
|Url NoLinks |Number of links in the email body. |N|
|Url NoIntLinks|Number of links whose target is internal to the email body.|N|
|Url NoExtLinks| Number of links whose target is outside the email body. |N|
|Url NoImgLinks|Number of links where the user needs to click on an image in body.|N|
|Url MaxNoPeriods|Number of periods in the link with the highest number of periods.|N|
|Url LinkText|If the human-readable link text contains link texts.|{0,1}|
|Url NonModalHereLinks|If the captured domain links to a domain other than the modal domain.|{0,1}|
| Url NoPorts |Number of links in the email that contain port information in the address. |N|
| Url WithFileExtensionNum |Number of URLs with a file extension. |N|
|Script Scripts                          | If scripts are used in the email body. |{0,1}|
|Script JavaScript                       | If javascript is used in the email body.   |{0,1}|
|Script StatusChange                     | If the script attempts to overwrite the status bar in the email client. |{0,1}|
|Script Popups                           | If the email contains pop-up window code. |{0,1}|
|Script NoOnClickEvents                  | Number of onClick events in the email.|N|
| Script NonModalJsLoads                  |If external javascript forms that come from modal domains.|{0,1}|
|Attachment Num                          | Number of attachments in the email.                |N|
|Attachment Doc Num                     | Number of 'doc' and 'docx' attachments in the email. |N|
| Attachment Xls Num                     | Number of 'xls' and 'xlsx' attachments in the email. |N|
| Attachment Ppt Num                    | Number of 'ppt' and 'pptx' attachments in the email.  |N|
|Attachment Pdf Num                     | Number of 'pdf' attachments in the email.|N|
|Attachment Exe Num                     | Number of 'exe' attachments in the email.|N|
|Attachment Image Num                  | Number of image attachments in the email.(jpg, jpeg, gif, bmp, png)|N|
|Attachment Video Num                   | Number of video attachments in the email.(avi, mkv, flv, mp4, mov)|N|
|Attachment Audio Num                   | Number of audio attachments in the email.(wav, mp3, aac, wma, ape)|N|
|Attachment Compressed Num              | Number of compressed file attachments in the email.(7z, zip, rar)|N|
|Attachment Text Num                    | Number of text attachments in the email.(txt, md, sh, bat, bak, html, htm)|N|
| Attachment Other Num                   | Number of other type attachments in the email.|N|

Notes:

(1)Body Richness = Body NumWords \ Body NumCharacters 

(2)Link texts contain click here, login and update.
