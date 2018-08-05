## **带您玩转Lambda，轻松构建Serverless后台【下篇】** 

## LambdaAuthorizerAPIRequest

```javascript
var async   = require('async');
var request = require('request');
 
// The API credentials of your service. These are needed to call Authlete
// Web APIs.
var api_key    = 'XXXXX';
var api_secret = 'XXXXXXXXXXXXXXXXXXXXXXXXXX';
 
// A function to call Authlete's introspection API.
// This function is used as a task for 'waterfall' method of 'async' module.
// See https://github.com/caolan/async#user-content-waterfalltasks-callback
// for details about 'waterfall' method.
//
//   * api_key (string) [REQUIRED]
//   * api_secret (string) [REQUIRED]
//       The API credentials of a service.
//
//   * access_token (string) [REQUIRED]
//       An access token whose information you want to get.
//
//   * scopes (string array) [OPTIONAL]
//       Scopes that should be covered by the access token. If the scopes
//       are not covered by the access token, the value of 'action' in the
//       response from Authlete's introspection API is 'FORBIDDEN'.
//
//   * subject (string) [OPTIONAL]
//       A subject (= unique identifier) of an end-user who should be
//       associated with the access token. If the subject is not associated
//       with the access token, the value of 'action' in the response from
//       Authlete's introspection API is 'FORBIDDEN'.
//
//   * callback
//       A callback function that 'waterfall' of 'async' module passes to
//       a task function.
//     
function introspect(api_key, api_secret, access_token, scopes, subject, callback)
{
  request({
    // The URL of Authlete's introspection API.
    url: 'https://api.authlete.com/api/auth/introspection',
 
    method: 'POST',
 
    // The API credentials of a service.
    auth: {
      username: api_key,
      pass: api_secret
    },
 
    // Request parameters for Authlete's introspection API.
    json: true,
    body: {
      token: access_token,
      scopes: scopes,
      subject: subject
    },
 
    // Interpret the response from Authlete's introspection API as a UTF-8 string.
    encoding: 'utf8'
  }, function(error, response, body) {
    if (error) {
      // Failed to call Authlete's introspection API.
      callback(error);
    }
    else if (response.statusCode != 200) {
      // The response from Authlete's introspection API indicates something wrong
      // has happened.
      callback(response);
    }
    else {
      // Call the next task of 'waterfall'.
      //
      // 'body' is already a JSON object. This has been done by 'request' module.
      // As for properties that the JSON object has, see the JavaDoc of
      // com.authlete.common.dto.IntrospectionResponse class in authlete-java-common.
      //
      //   http://authlete.github.io/authlete-java-common/com/authlete/common/dto/IntrospectionResponse.html
      // 
      callback(null, body);
    }
  });
}
 
// The main code to be performed after the access token is validated.
//
//   * event
//       The 'event' parameter given to the 'handler' function.
//
//   * context
//       The 'context' parameter given to the 'handler' function.
//
//   * info
//       A JSON object that represents the response from Authlete's introspection API.
//       As for properties that the JSON object has, see the JavaDoc of
//       com.authlete.common.dto.IntrospectionResponse class in authlete-java-common.
//
//         http://authlete.github.io/authlete-java-common/com/authlete/common/dto/IntrospectionResponse.html
//       
function main(event, context, info)
{
  // Returns a JSON.
  context.succeed({

    "Hello": "API Gateway+Lambda with an OAuth 2.0 access token on AWS China region.",
 
    // The ID of the client application that is associated with the access token.
    "clientId": info.clientId,
 
    // The subject (= unique identifier) of the end-user that is associated with
    // the access token. This is not available if the access token has been created
    // by using 'Client Credentials Flow' (RFC 6749, 4.4. Client Credentials Grant).
    "subject": info.subject,
 
    // The scopes (= permissions) that are associated with the access token.
    // This may be null.
    "scopes": info.scopes
  });
}
 
// The entry point of this lambda function.
exports.handler = function(event, context) {
  // [string] The access token that the client application presented.
  // The value comes from the request parameter 'access_token'.
  var access_token = event.access_token;
 
  // [string array] Scopes (= permissions) that this REST API requires.
  // In this example, no scope is required.
  var scopes = null;
 
  // [string] The subject (= unique identifier) of the end-user that
  // this REST API expects the access token to be associated with.
  // In this example, no specific subject is required.
  var subject = null;
 
  // Validate the access token and then invoke the main code.
  async.waterfall([
    function(callback) {
      // Validate the access token by calling Authlete's introspection API.
      introspect(api_key, api_secret, access_token, scopes, subject, callback);
    },
    function(response, callback) {
      switch (response.action) {
        case 'OK':
          // The access token is valid. Execute the main code.
          main(event, context, response);
          break;
 
        default:
          // The access token is not valid or another different problem
          // has happened. The format of the error message we are about
          // to construct is as follows.
          //
          //     {action}:{resultMessage}
          //
          // Possible values that {action} takes here are as follows:
          //
          //     BAD_REQUEST
          //     UNAUTHORIZED
          //     FORBIDDEN
          //     INTERNAL_SERVER_ERROR
          //
          // This error message format is important because its regular
          // expression is referred to in 'Integration Response' section
          // of Amazon API Gateway.
          context.fail(response.action + ":" + response.resultMessage);
          break;
      }
 
      callback(null);
    }
  ], function (error) {
    if (error) {
      // Something wrong has happened.
      context.fail(error);
    }
  });
};
```

## LambdaCloudWatchScheduledEventTest

```python
import boto3
# Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g., 'us-east-1'
region = 'cn-north-1'
# Enter your instances here: ex. ['X-XXXXXXXX', 'X-XXXXXXXX']
instances = ['i-0b8b7e28420b82ca6']

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name=region)
    ec2.stop_instances(InstanceIds=instances)
    print 'stopped your instances: ' + str(instances)
```

## LambdaConcurrentExecutionsDemo

```javascript
exports.handler = (event, context, callback) => {
    // TODO implement
    setTimeout(function() {
        console.log('wait me some time');
    }, 1000);
    callback(null, 'Hello from Lambda');
};

```

## LambdaEC2AutoTag

```python
from __future__ import print_function
import json
import boto3
import logging
import time
import datetime

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):
    #logger.info('Event: ' + str(event))
    #print('Received event: ' + json.dumps(event, indent=2))

    ids = []

    try:
        region = event['region']
        detail = event['detail']
        eventname = detail['eventName']
        arn = detail['userIdentity']['arn']
        principal = detail['userIdentity']['principalId']
        userType = detail['userIdentity']['type']

        if userType == 'IAMUser':
            user = detail['userIdentity']['userName']

        else:
            user = principal.split(':')[1]

        logger.info('principalId: ' + str(principal))
        logger.info('region: ' + str(region))
        logger.info('eventName: ' + str(eventname))
        logger.info('detail: ' + str(detail))

        if not detail['responseElements']:
            logger.warning('Not responseElements found')
            if detail['errorCode']:
                logger.error('errorCode: ' + detail['errorCode'])
            if detail['errorMessage']:
                logger.error('errorMessage: ' + detail['errorMessage'])
            return False

        ec2 = boto3.resource('ec2')

        if eventname == 'CreateVolume':
            ids.append(detail['responseElements']['volumeId'])
            logger.info(ids)

        elif eventname == 'RunInstances':
            items = detail['responseElements']['instancesSet']['items']
            for item in items:
                ids.append(item['instanceId'])
            logger.info(ids)
            logger.info('number of instances: ' + str(len(ids)))

            base = ec2.instances.filter(InstanceIds=ids)

            #loop through the instances
            for instance in base:
                for vol in instance.volumes.all():
                    ids.append(vol.id)
                for eni in instance.network_interfaces:
                    ids.append(eni.id)

        elif eventname == 'CreateImage':
            ids.append(detail['responseElements']['imageId'])
            logger.info(ids)

        elif eventname == 'CreateSnapshot':
            ids.append(detail['responseElements']['snapshotId'])
            logger.info(ids)
        else:
            logger.warning('Not supported action')

        if ids:
            for resourceid in ids:
                print('Tagging resource ' + resourceid)
            ec2.create_tags(Resources=ids, Tags=[{'Key': 'Owner', 'Value': user}, {'Key': 'PrincipalId', 'Value': principal}])

        logger.info(' Remaining time (ms): ' + str(context.get_remaining_time_in_millis()) + '\n')
        return True
    except Exception as e:
        logger.error('Something went wrong: ' + str(e))
        return False
```

## LambdaKinesisDemo

```javascript
'use strict';
console.log('Loading function');
var AWS = require('aws-sdk'); 
var docClient = new AWS.DynamoDB.DocumentClient(); 
AWS.config.region = 'cn-north-1';
exports.handler = function(event, context, callback) {
    event.Records.forEach(function(record) {
        // Kinesis data is base64 encoded so decode here
        var payload = new Buffer(record.kinesis.data, 'base64').toString('ascii');
        console.log('Decoded payload:', payload);
        var params = { 
          Item: { 
               date: Date.now(), 
               message: payload 
          }, 
          TableName: 'Lambda-DynamoDB-Write-Read' 
        }; 
        docClient.put(params, function(err, data){ 
            if(err){ 
                console.log("Fail to Write into AWS DynamoDB.");
                callback(err, null); 
            }else{ 
                console.log("Sucessfully write item into AWS DynamoDB.");
                callback(null, data); 
            } 
         }); 
    });
    callback(null, "message");
};
```

## LambdaPostCommentHTML

```javascript
'use strict';
var fs = require('fs');
exports.handler = function(event, context) {
    var contents = fs.readFileSync("public/index.html");
    context.succeed({
        statusCode: 200,
        body: contents.toString(),
        headers: { 'Content-Type': 'text/html' }
    });
};
```

```html
<!DOCTYPE html>
<html>
<head>
 <meta http-equiv="content-type" content="text/html; charset=GBK">
 <title>LambdaAPIGatewanyDynamoDB</title>
 <script type="text/javascript" src="https://s3.cn-north-1.amazonaws.com.cn/bjsdemo/LambdaAPIGWComments/jquery-2.1.3.min.js"></script>
</head>

<body>
 <h1>AWS China region (cn-north-1)</h1>
 <h2>Lambda + API Gateway + DynamoDB + S3</h2>
 <h1>Loading all comment items</h1>
 <div id="entries">

 </div>
 <h1>Write new comment</h1>
 <form>
  <label for="message">Message</label>
  <textarea id="message"></textarea>
  <button id="submitbutton">Submit Comment</button>
 </form>

 <script type="text/javascript">
  var API_Gateway_URL = 'https://ihqwdda7ad.execute-api.cn-north-1.amazonaws.com.cn/prod/danrong';

  $(document).ready(function() {
   $.ajax({
    type: 'GET',
    url: API_Gateway_URL,

    success: function(data) {
     $('#entries').html('');
     data.Items.forEach(function(getcomments) {
      $('#entries').append('<p>' + getcomments.message + '</p>');
     })
    }
   });
  });

  $('#submitbutton').on('click', function() {
   $.ajax({
    type: 'POST',
    url: API_Gateway_URL,
    data: JSON.stringify({ "message": $('#message').val() }),
    contentType: "application/json",
    success: function(data) {
     location.reload();
    }
   });
   return false;
  });
 </script>
</body>
</html>
```

## LambdaS3CreateThumbnail

```javascript
// dependencies
var async = require('async');
var AWS = require('aws-sdk');
var gm = require('gm').subClass({ imageMagick: true }); // Enable ImageMagick integration.
var util = require('util');

// constants
var MAX_WIDTH  = 100;
var MAX_HEIGHT = 100;

// get reference to S3 client 
var s3 = new AWS.S3();
 
exports.handler = function(event, context, callback) {
    // Read options from the event.
    console.log("Reading options from event:\n", util.inspect(event, {depth: 5}));
    var srcBucket = event.Records[0].s3.bucket.name;
    // Object key may have spaces or unicode non-ASCII characters.
    var srcKey    = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));
    //var dstBucket = srcBucket + "-resized";
    var dstKey    = "resized-" + srcKey;

    // // Sanity check: validate that source and destination are different buckets.
    // if (srcBucket == dstBucket) {
    //     callback("Source and destination buckets are the same.");
    //     return;
    // }

    // Infer the image type.
    var typeMatch = srcKey.match(/\.([^.]*)$/);
    if (!typeMatch) {
        callback("Could not determine the image type.");
        return;
    }
    var imageType = typeMatch[1];
    if (imageType != "jpg" && imageType != "png") {
        callback('Unsupported image type: ${imageType}');
        return;
    }

    // Download the image from S3, transform, and upload to a different S3 bucket.
    async.waterfall([
        function download(next) {
            // Download the image from S3 into a buffer.
            s3.getObject({
                    Bucket: srcBucket,
                    Key: srcKey
                },
                next);
            },
        function transform(response, next) {
            gm(response.Body).size(function(err, size) {
                // Infer the scaling factor to avoid stretching the image unnaturally.
                var scalingFactor = Math.min(
                    MAX_WIDTH / size.width,
                    MAX_HEIGHT / size.height
                );
                var width  = scalingFactor * size.width;
                var height = scalingFactor * size.height;

                // Transform the image buffer in memory.
                this.resize(width, height)
                    .toBuffer(imageType, function(err, buffer) {
                        if (err) {
                            next(err);
                        } else {
                            next(null, response.ContentType, buffer);
                        }
                    });
            });
        },
        function upload(contentType, data, next) {
            // Stream the transformed image to a different S3 bucket.
            s3.putObject({
                    Bucket: srcBucket,
                    Key: dstKey,
                    Body: data,
                    ContentType: contentType
                },
                next);
            }
        ], function (err) {
            if (err) {
                console.error(
                    'Unable to resize ' + srcKey +
                    ' and upload to ' + srcBucket + '/' + dstKey +
                    ' due to an error: ' + err
                );
            } else {
                console.log(
                    'Successfully resized ' + srcBucket + '/' + srcKey +
                    ' and uploaded to ' + srcBucket + '/' + dstKey
                );
            }
            callback(null, "message");
        }
    );
};
```

## LambdaSNSNotification

```javascript
'use strict'; 
console.log('Loading function'); 
var AWS = require('aws-sdk'); 
var docClient = new AWS.DynamoDB.DocumentClient(); 
AWS.config.region = 'cn-north-1';
exports.handler = function (event, context, callback){ 
     console.log("AWS China Lambda and SNS Notification");
     const message = event.Records[0].Sns.Message;
     console.log("From SNS:", message);
     var params = {
          Item: { 
               date: Date.now(), 
               message: message 
          }, 
          TableName: 'Lambda-DynamoDB-Write-Read' 
     }; 
     docClient.put(params, function(err, data){ 
          if(err){ 
               console.log("Fail to Write into AWS DynamoDB.");
               callback(err, null); 
          }else{ 
               console.log("Sucessfully write item into AWS DynamoDB.");
               callback(null, data); 
          } 
     }); 
}
```

## LambdaSQSMessageTrigger

```python
import json
def lambda_handler(event, context):
  body = {
      "message": "Hello SQS",
      "event": event
  }
  print json.dumps(body)

  response = {
      "statusCode": 200,
      "body": json.dumps(body)
  }
  return response
```

