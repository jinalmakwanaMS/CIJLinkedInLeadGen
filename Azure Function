const https = require('https');
module.exports = async function (context, req) {
    context.log('HTTP trigger function processed a request.');

    if (req.method === 'GET') {
        // Handle GET request
        context.log('-----------------GET-----------------');
        context.log(req);
        const crypto = require('crypto');
        const challengeCode = req.query.challengeCode;
        const clientSecret = req.query.clientSecret || "<<Your Client Secret>>";
        const hmac = crypto.createHmac('sha256', clientSecret);
        hmac.update(challengeCode);
        const challengeResponse = hmac.digest('hex')
        const res = {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: { 
                challengeCode: challengeCode,
                challengeResponse: challengeResponse 
                }
        };
        context.log(res);
        context.res = res;        

    } else if (req.method === 'POST') {
        context.log('-----------------POST.-----------------');
        context.log(req);
        const res = {body: req.body};
        context.log('Request Body:', res);
        
        //context.res = res;        
        
        // Extract the leadGenFormResponse value from the request body
        const leadGenFormResponse = req.body.leadGenFormResponse;
        
        if (!leadGenFormResponse) {
            context.res = {
                status: 400,
                body: "Missing required parameter."
            };
            return;
        }
        
        // Log the leadGenFormResponse value
        context.log('Extracted leadGenFormResponse:', leadGenFormResponse);
        // Remove the "urn:li:leadGenFormResponse:" prefix if present
        const leadGenFormResponseId = leadGenFormResponse.replace('urn:li:leadGenFormResponse:', '');
        // Log the modified leadGenFormResponse ID
        context.log('Modified leadGenFormResponse:', leadGenFormResponseId);
        context.log('-----------------Trigger Power Automate-----------------');
        // URL to trigger your Power Automate Flow 
        const flowUrl = "<<Your HTTP URL from Power Automate Flow>>";

        // The request body that will be passed to Power Automate Flow
        const requestBody = JSON.stringify({
            "leadGenFormResponse": leadGenFormResponseId   // Adjust the parameter name according to what your flow expects
        });
        context.log('Request Body (stringified) for PA:', requestBody);
        // Options for the POST request (no need for authorization if the flow doesn't require it)
        const options = {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Content-Length': Buffer.byteLength(requestBody)
            }
        };

        // Send the request to trigger the flow
        const reqToFlow = https.request(flowUrl, options, (res) => {
            let data = '';

            res.on('data', (chunk) => {
                data += chunk;
            });

            res.on('end', () => {
                context.log('Flow triggered successfully with status:', res.statusCode);
                if (res.statusCode !== 200) {
                    context.log('Error in triggering flow:', data);
                }
                context.res = {
                    status: res.statusCode,
                    body: data,
                };
            });
        });

        reqToFlow.on('error', (error) => {
            context.log('Error occurred while triggering Power Automate flow:', error);
            context.res = {
                status: 500,
                body: 'Failed to trigger flow',
            };
        });

        // Send the POST data to trigger the flow
        context.log('Request Body (stringified) for PA:', requestBody);
        reqToFlow.write(requestBody);
        reqToFlow.end();
        

        /*context.log('-----------------Get Form Response within Azure Function.-----------------');
        
            // Define the URL to fetch data from LinkedIn API
            const url = 'https://api.linkedin.com/rest/leadFormResponses/'+leadGenFormResponseId +'?&fields=ownerInfo,associatedEntityInfo,leadMetadataInfo,owner,leadType,versionedLeadGenFormUrn,id,submittedAt,testLead,formResponse,form:(hiddenFields,creationLocale,name,id,content)';            
            // Set the headers for the request
            const headers = {
                'LinkedIn-Version': '202401',
                'Authorization': 'Bearer <<Your Authorization Token>>', // Replace with your actual access token
                'X-Restli-Protocol-Version': '2.0.0',
                'Content-Type': 'application/json'
            };

            const fetchData = () => {
            return new Promise((resolve, reject) => {
                const req = https.request(url, { method: 'GET', headers }, (res) => {
                    let data = '';

                    // Accumulate data chunks
                    res.on('data', (chunk) => {
                        data += chunk;
                    });

                    // Resolve the Promise with the response data on end
                    res.on('end', () => {
                        resolve(JSON.parse(data));
                    });
                });

                req.on('error', (error) => {
                    reject(error);
                });

                req.end();
            });
        };

        try {
            const responseData = await fetchData();
            context.log('Full Response Data:', JSON.stringify(responseData, null, 2)); // Use JSON.stringify to expand objects
		
		// call Dataverse API to create lead from the parse JSON lead response data
            // Set the response for the Azure Function to return it as-is
            context.res = {
                status: 200,
                body: responseData
            };
        } catch (error) {
            context.log('Error occurred:', error.message);

            context.res = {
                status: 500,
                body: { message: 'An error occurred', error: error.message }
            };
        }*/
        
    } else {
        context.res = {
            status: 405,
            body: "Method Not Allowed"
        };
    }
};
