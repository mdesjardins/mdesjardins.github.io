---
title: Using JavaMail to read and extract attachments
author: mdesjardins
layout: post
permalink: /2008/03/13/using-javamail-to-read-and-extract/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/03/using-javamail-to-read-and-extract.html
categories:
  - blog
tags:
  - java
  - javamail
---
Recently I had to create a JavaMail-based e-mail client that polled an IMAP server for incoming e-mail messages. The client needed to read each message, find any attachments, and save those attachments to a directory on the application server. I had never had a need to do this with the JavaMail API before, so I did the first thing that many developers do when they&#8217;re venturing into unknown territory, and Google&#8217;d for an example. I found lots of examples of sending attachments, but not many of receiving them. The few examples that I did find of receiving mail with the JavaMail API were too simple. So I had to figure out how to do it by (gasp) reading the documentation. Fortunately, it wasn&#8217;t very difficult, but I thought I&#8217;d share it for the next programmer that turns to Google for a solution.

In my application, I used the [Quartz][1] library to periodically call a method named <span style="font-weight: bold;">receive</span>, which does the grunt work of pulling stuff off the server. This code is similar to other examples you might find out there Google&#8217;ing, and it&#8217;s pretty mundane stuff. The MailReceiveException is a catch-all Exception class that I wrote:

``` java
public Message[] receive(String server, String user,
                         String password, String directory)
                 throws MailReceiveException {
  Message[] msgs;
  Store store = null;
  Folder folder = null;
  try {
    store = session.getStore("imap");
    store.connect(server, user, password);
    folder = store.getDefaultFolder();
    if (folder == null) {
      throw new MailReceiveException("No default folder");
    }
    folder = folder.getFolder("INBOX");
    if (folder == null) {
      throw new MailReceiveException("No IMAP INBOX");
    }
    folder.open(Folder.READ_WRITE);
    msgs = folder.getMessages();
    for (int i=; i<msgs.length; ++i) {
      Message msg = msgs[i];
      // don't fetch messages that we've already processed
      if (!msg.isSet(Flags.Flag.SEEN)) {
        String from = "unknown";
        if (msg.getReplyTo().length >= 1) {
          from = msg.getReplyTo()[].toString();
        } else if (msg.getFrom().length >= 1) {
          from = msg.getFrom()[].toString();
        }
        String subject = msg.getSubject();
        String filename = directory + "/" + from + ": " + subject;
        // This is where the real work will get done.
        this.saveParts(msg.getContent(), filename);
        msg.setFlag(Flags.Flag.SEEN,true);
      } else {
        // Delete anything that is more than sixty days old.
        Date receivedOn = msg.getReceivedDate();
        GregorianCalendar cal = new GregorianCalendar();
        cal.add(GregorianCalendar.DATE,-60);
        if (receivedOn.before(cal.getTime())) {
          msg.setFlag(Flags.Flag.DELETED, true);
        }
      }
    }
 
    // Rats' nest of catch blocks omitted... make sure you close
    // the folder and the mail store in a finally block, as well
    // as expunge any messages that have been marked for deletion.
 
  }
  return msgs;
}
```

There&#8217;s one small detail that I had to learn the hard way, and that is that the &#8220;Parts&#8221; within Multipart MIME messages can themselves contain other Parts. So you have a nifty opportunity to use some of that fancy recursion stuff that you learned about in your &#8220;Intro to Programming&#8221; course! 

``` java
public void saveParts(Object content,
                       String filename)
             throws MailReceiveException {
  OutputStream out = null;
  InputStream in = null;
  try {
    if (content instanceof Multipart) {
      Multipart multi = ((Multipart)content);
      int parts = multi.getCount();
      for (int j=; j<parts; ++j) {
        MimeBodyPart part = (MimeBodyPart)multi.getBodyPart(j);
        if (part.getContent() instanceof Multipart) {
          // part-within-a-part, do some recursion...
          this.saveParts(part.getContent(), filename);
        } else {
          String type = part.getContentType();
          String extension = "";
          if (part.isMimeType("text/html")) {
            extension = "html";
          } else {
            if (part.isMimeType("text/plain")) {
              extension = "txt";
            } else {
              // Try to make a reasonable guess about the
              // extension from the MIME type.
              int end = type.indexOf(';');
              if (end < ) {
                end = type.length();
              }
              extension = type.substring(type.indexOf('/')+1,end);
            }
            filename = filename + "." + extension;
            out = new FileOutputStream(new File(filename));
            in = part.getInputStream();
            int k;
            while ((k = in.read()) != -1) {
              out.write(k);
            }
          }
        }
      }
    }
  } catch (MessagingException e1) {
    log.error("Messaging Exception", e1);
    throw new MailReceiveException("Error fetching e-mail");
  } catch (IOException e2) {
    log.error("Caught IOException reading e-mail.", e2);
    throw new MailReceiveException("Error fetching e-mail");
  } finally {
    if (in != null) {
      try {
        in.close();
      } catch (IOException e6) {
        log.error("Unable to close input stream");
      }
    }
    if (out != null) {
      try {
        out.flush();
        out.close();
      } catch (IOException e7) {
        log.error("Unable to close output stream.");
      }
    }
  }
}
```

All of this work was done using JavaMail 1.3. I believe newer versions of the API have a <span style="font-weight: bold;">savePart</span> method, so you don&#8217;t need to write out the parts to a FileOutputStream byte-by-byte as I have above.

 [1]: http://www.opensymphony.com/quartz/