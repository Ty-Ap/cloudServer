# cloudServer


deployed fine, but could eb init. tried a lot of fixes with TA and by myself. got this error and some others

![uml](./assets/Screenshot%20(328).png)

[link](http://cloudserver-env.eba-yv29pf3z.us-east-1.elasticbeanstalk.com/)


figured out eb stuff, and debugged.

this is some of what i tried on lab, // import { S3Client , GetObjectCommand, putObjectCommand} from '@aws-sdk/client-s3';

// export const handler = async(event) => {
//     console.log('image uploaded');
//     console.log('image details', event.Records[0].s3);
//     const response = {
//         statusCode: 200,
//         body: JSON.stringify('Hello from Lambda!')
//     };
//     return response;
// };

const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event, context) => {
  const bucketName = 'tty';
  const fileName = 'images.json';

  // Download the images.json file from the S3 bucket if it exists
  let imagesArray = [];
  try {
    const s3Object = await s3.getObject({ Bucket: bucketName, Key: fileName }).promise();
    imagesArray = JSON.parse(s3Object.Body.toString());
  } catch (error) {
    if (error.code !== 'NoSuchKey') {
      console.error(`Error downloading ${fileName} from S3:`, error);
      return;
    }
  }

  // Create a metadata object for a new image
  const newImageMetadata = {
    name: 'example.jpg',
    size: 1024,
    type: 'image/jpeg',
    // ... other metadata properties
  };

  // Check if the image already exists in the array
  const existingImageIndex = imagesArray.findIndex(image => image.name === newImageMetadata.name);

  if (existingImageIndex === -1) {
    // Add the new image to the array
    imagesArray.push(newImageMetadata);
  } else {
    // Update the existing image's metadata
    imagesArray[existingImageIndex] = newImageMetadata;
  }

  // Upload the updated images.json file back to the S3 bucket
  try {
    const s3Params = {
      Bucket: bucketName,
      Key: fileName,
      Body: JSON.stringify(imagesArray),
      ContentType: 'application/json',
      ACL: 'private',
    };
    await s3.putObject(s3Params).promise();
  } catch (error) {
    console.error(`Error uploading ${fileName} to S3:`, error);
    return;
  }
};


