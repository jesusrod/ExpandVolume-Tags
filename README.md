# ExpandVolume-Tags
Expand EBS Volumes based on a max size set on a tab - 'ebsmax'

Copyright (c) 2021. Amazon Web Services Ireland LTD. All Rights Reserved. AWS Enterprise Support Team Name : ExpandVolume-Tags Version : 1.0.0 Encoded : No Paramaters : None Platform : Any EC2 Associated Files: None Abstract : ReadMe Document Revision History : - 1.0.0 | 16/03/2021 Initial Version Author : Jesus Rodriguez Cisneros

############################################################## ExpandVolume-Tags .################################################################################## YOU ACKNOWLEDGE AND AGREE THAT THE SCRIPT IS PROVIDED FREE OF CHARGE "AS IS" AND "AVAILABLE" BASIS, AND THAT YOUR USE OF RELIANCE UPON THE APPLICATION OR ANY THIRD PARTY CONTENT AND SERVICES ACCESSED THEREBY IS AT YOUR SOLE RISK AND DISCRETION. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

####################################################################################################################################################################

This Automation is part of a deployment that aims to expand ebs volumes over EC2-Instances when their C:\ volume free disk space falls below 10%. The automation will review whether the EC2-Instance has a tag called 'ebsmax' and will read the value which is the defined maximum EBS Volume size. Therefore it will calculate and expand the volume to 20% of it's current size if it's below the max volume size read from the tag. Before expanding the volume it will create an EBS Snapshot and will tag it. Once the volume is expanded it will expand the Windows partition and publish a message over SNS to a topic previously defined.

The only parameter required is the EC2-InstanceID.

If the ebs size increase is over the max value defined on the tag, then the automation will not increase the volume and will publish a message from the step called PublishSNSNotificationfail.

Step name
1	 getInstanceDetails # read the root volumeid
2	 getRootVolumeId # current ebs size
3	 ReadInstanceTag # reading the EC2 tag with 'ebsmax' as the Key Name and will retrieve the value which the max numeric size (this needs to be just numeric i.e. 300)
4	 CalculateIncrease # it will calculate a 20% from the current ebs volume size.
5	 evaluateifincrease # it will compare the current size if the 20% increase will surpass the Ebsmax tag value read previously
6	 createSnapshot # create EBS Snapshot before increasing
7	 ModifyVolumeSize # increase EBS volume with the calculated increase size
8	 ExpandVolume # Expand the EBS volume in Windows only
9	 createTags # it will create a tag on the EBS Snapshot recently created.
10 PublishSNSNotification # send SNS message to a topic hardcoded - if you want a paramenter shot me a message.
11 PublishSNSNotificationfail # send SNS message if the increase size is over the tag value, or if the tag is missing from the EC2 instance.

This deployment is best combined with CloudWatch Alarms and triggering a SNS notification which can trigger the Automation from Event Bridge. 
Another option is to use AWS Config and trigger a Lambda function that can trigger this automation passing the EC2 parameter from the SNS topic from example.
Finally, this can be trigger manually from Systems Management and input the InstanceID.

I'll focus on this integration with a CloudWatch Alarm that is triggered from CloudWatch custom metric to capture the C: drive free disk space (can be configured from AWS-ConfigureCloudWatch automation for example). I'm adding Event Brigde event pattern example to monitor when the CW alarm is on the "Alarm" state.
