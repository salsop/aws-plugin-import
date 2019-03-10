import boto3
import ssl
import urllib
import urllib.parse
import urllib.request
import xml
import xml.etree.ElementTree

panorama_hostname = 'PANORAMA.LOCAL'
panorama_username = 'ManagementUser'
panorama_password = 'Passw0rd!'
aws_iam_role = 'IAM ROLE'
aws_notify_group = 'AWS VMs'

def makeApiCall(hostname,data):
    # Context to separate function?
    # check response for status codes and return reponse.read() if success
    #   Else throw exception and catch it in calling function
    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    url = "https://" + hostname + "/api"
    encoded_data = urllib.parse.urlencode(data).encode('utf-8')
    return urllib.request.urlopen(url, data=encoded_data, context=ctx).read()

def panSetConfig(hostname, api_key, xpath, element):
    data = {
        'type': 'config',
        'action': 'set',
        'key': api_key,
        'xpath': xpath,
        'element': element
    }
    response = makeApiCall(hostname, data)
    # process response and return success or failure?
    # Debug should print output as well?
    if xml.etree.ElementTree.XML(response)[0].text == 'command succeeded':
        return "success"
    else:
        return "failure"

def create_aws_monitoring_def(hostname, api_key, vpcid, iamrole, location, notifygroup):
    print(" > create_aws_montioring_def: " + vpcid)
    xpath = "/config/devices/entry/plugins/aws/monitoring-definition/entry[@name='" + vpcid + "']"
    element = "<vpc-id>" + vpcid + "</vpc-id>"\
                  "<iam-role>" + iamrole + "</iam-role>"\
                  "<endpoint-url>ec2." + location + ".amazonaws.com</endpoint-url>"\
                  "<notify-group>" + notifygroup + "</notify-group>"
    return panSetConfig(hostname, api_key, xpath, element)

def getApiKey(hostname, username, password):
    data = {
        'type' : 'keygen',
        'user' : username,
        'password' : password
    }
    response = makeApiCall(hostname, data)
    return xml.etree.ElementTree.XML(response)[0][0].text

def panCommit(hostname, api_key, message=""):
    '''Function to commit configuration changes
    '''
    data = {
        "type" : "commit",
        "key" : api_key,
        "cmd" : "<commit>{0}</commit>".format(message)
    }
    return makeApiCall(hostname, data)



panorama_apikey = getApiKey(panorama_hostname, panorama_username, panorama_password)
ec2client = boto3.client('ec2')
for region in ec2client.describe_regions()['Regions']:
    ec2 = boto3.resource('ec2', region_name=region['RegionName'])
    ec2client = boto3.client('ec2', region_name=region['RegionName'])
    response = ec2client.describe_vpcs()
    for vpc in response['Vpcs']:
        create_aws_monitoring_def(panorama_hostname, panorama_apikey, vpc['VpcId'], aws_iam_role, region['RegionName'], aws_notify_group)
panCommit(panorama_hostname, panorama_apikey)
