# serverless_OpenPIV
An idea to use AWS Lambdas or Google Cloud functions as a Serverless Framework for super-fast PIV processing in the cloud

## Option 1
1. User uploads N image pairs (`001_a.tif`, `001_b.tif`, `002_a.tif`, `002_b.tif`...) to a S3 bucket
2. Lambda function detects the upload and triggers the processing of `openpiv_python` on every new pair and stores the result 
in the results bucket

The limitation is that a single pair processing should have less than a time limit allowed for lambda and this might fail for multi-pass and window deformation complex analysis types. 

## Option 2
1. Upload of a new pair of images trigger the lambda `splitter`
2. `splitter` reads the pair and splits it into sub-windows (interrogation windows). Every sub-window is stored as a pair of small images into a separate bucket (from a first pair we get many pairs of sub-windows in mutliple buckets. It's possible also that the split won't be identical, i.e. it is useful of sub-window from image B is larger and covers a larger area around the sub-window in image A.
3. `analyzer` is another lambda function that watches the small sub-windows bucket and triggers the analysis with `openpiv-python` on a single sub-window. So we trigger as many lambdas as number of sub-windows. This process should be super-fast and efficient. `analyzer` stores the results in the results bucket - either adds a row to the same ASCII file or appends the pandas DataFrame. It is also efficient as we do not care in which order the rows will be written, i.e. which lambda will finish first. But it has to be efficient in writing so lambdas do not wait to write because of others. So maybe we better write into separate files and trigger another lambda `collector` 
4. `collector` watches all the results created by `analyzer`s and combines them into a single file that is sent to the user bucket. 


## Additional ideas to discuss
![Additional ideas and links](links_and_ideas.md)
