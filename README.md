```javascript
import nodemailer from 'nodemailer';
import dotenv from 'dotenv';
import { globalConfig } from '../config';
import { compile } from 'handlebars';

dotenv.config();

class EmailService {
  constructor() {
    this.transporter = nodemailer.createTransport({
      host: process.env.EMAIL_HOST,
      port: process.env.EMAIL_PORT,
      secure: process.env.EMAIL_SECURE,
      auth: {
        user: process.env.EMAIL_USERNAME,
        pass: process.env.EMAIL_PASSWORD
      }
    });
  }

  async sendEmail(options) {
    try {
      const info = await this.transporter.sendMail(options);
      console.log(`Email sent: ${info.response}`);
      return info;
    } catch (error) {
      console.error(error);
      throw error;
    }
  }

  async send(options) {
    const defaultOptions = {
      from: process.env.EMAIL_FROM
    };

    const mergedOptions = { ...defaultOptions, ...options };

    if (mergedOptions.template) {
      const template = compile(mergedOptions.template);
      mergedOptions.html = template(mergedOptions.data);
      delete mergedOptions.template;
      delete mergedOptions.data;
    }

    if (mergedOptions.attachments) {
      mergedOptions.attachments = mergedOptions.attachments.map((attachment) => ({
        filename: attachment.filename,
        content: attachment.content,
        contentType: attachment.contentType
      }));
    }

    return this.sendEmail(mergedOptions);
  }
}

export default new EmailService();



EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USERNAME=your-email@gmail.com
EMAIL_PASSWORD=your-password
EMAIL_FROM=your-email@gmail.com

import EmailService from './email.service';

const emailService = EmailService;

const template = `
  <p>Hello, {{name}}!</p>
  <p>Your order has been shipped.</p>
  <p>Order details:</p>
  <ul>
    {{#items}}
    <li>{{name}} x {{quantity}}</li>
    {{/items}}
  </ul>
`;

const data = {
  name: 'John Doe',
  items: [
    { name: 'Product 1', quantity: 2 },
    { name: 'Product 2', quantity: 1 }
  ]
};

emailService.send({
  to: 'recipient@example.com',
  subject: 'Order shipped',
  template,
  data
}).then((info) => {
  console.log(`Email sent: ${info.response}`);
}).catch((error) => {
  console.error(error);
});

```
