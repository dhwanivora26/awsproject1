provider "aws"{
 region="ap-south-1"
 access_key = "A*********************"
  secret_key = "Yn***&******************************"
}


resource "aws_key_pair" "firstkey"{
 key_name ="firstkey"
 public_key="ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAiESLPVyE31cywfuFaZDKjr0zZ/S1Xs78ntZCpas3OI3BMBJPel0KJ7qUyi3lCl2EEkpo4Iju07UsknjtfYvJ93NOrT7QoAunLN071fs4ifTIpB5vMdSTRU1Jj4FwL0Hf+vlHjdxaxi6qbFBVtdGVsrLl5ym8R2xrRF7eVBll1UjBHX/B/SbPMNWJCVZG4dQ9zIckjJ6aghyrJx+IYzkqDWaSIzAO7JsygyJ9TdWMdToLT6ViUWnweWLg5vBx9NzjzbCtTvaWoGnwFmmJ3aMsnk1svcTenJgF0IlvqcOp7T1C2E11TJSRza8tkZbpUOKRoXcnzOT7RiaAAj+KIAiXlw== rsa-key-20200615"
}


resource "aws_instance" "web1"{
 ami ="ami-0447a12f28fddb066"
 instance_type="t2.micro"
 key_name="firstkey"
 security_groups=["securitygrp"]
 connection{
  type="ssh"
  user ="ec2-user"
  private_key =file("C:/Users/Dhaval/Downloads/firstkey.pem")
  host =aws_instance.web1.public_ip
  }
}
resource "aws_security_group" "securitygrp"{
 name="mysecurity"
 description="Allow HTTP and SSH inbound traffic"
 vpc_id="vpc-0bbea363"
 ingress{ 
   description="HTTP"
   from_port =80
   to_port =80
   protocol ="tcp"
   cidr_blocks=["0.0.0.0/0"]
  }
 
  ingress{ 
   description="SSH"
   from_port =22
   to_port =22
   protocol ="tcp"
   cidr_blocks=["0.0.0.0/0"]
  }
  egress{
   from_port =0
   to_port =0
   protocol ="-1"
   cidr_blocks=["0.0.0.0/0"]
      }
  tags={
     Name="t1"
   }
}

resource "aws_ebs_volume" "ebs"{
  availability_zone=aws_instance.web1.availability_zone
  size =1
  tags={
   Name="mypendrive1"
  }
}

resource "aws_volume_attachment" "attachment"{
   device_name="/dev/sdh"
   instance_id=aws_instance.web1.id
   volume_id=aws_ebs_volume.ebs.id
   force_detach =true
}

resource "aws_s3_bucket" "storage1"{
   bucket="storage1"
   acl="public-read"
   
   tags={
   Name="mystorage"
   Environment="Dev"
   }
}

locals{
  s3_origin_id="myorigin"
}

resource"aws_s3_bucket_object" "obj"{
   depends_on= [
            aws_s3_bucket.storage1,
       ]
   bucket="storage1"
   key  ="awsimage1.jpg"
   source="C:/Users/Dhaval/Downloads/awsimage1.jpg"
   content_type="image/jpg"
   etag="C:/Users/Dhaval/Downloads/awsimage1.jpg"
   acl="public-read"
 }

resource "aws_cloudfront_distribution" "s3_distribution" {
 
 origin {
    domain_name = "aws_s3_bucket.storage1.bucket_domain_name"
    origin_id   = "local.s3_origin_id"
  }
 
 enabled =true
 default_root_object="awsimage1.jpg"

 default_cache_behavior {
    allowed_methods  = ["DELETE", "GET", "HEAD", "OPTIONS", "PATCH", "POST", "PUT"]
    cached_methods   = ["GET", "HEAD"]
    target_origin_id = "local.s3_origin_id"

    forwarded_values {
      query_string = false

      cookies {
        forward = "none"
      }
    }

    viewer_protocol_policy = "allow-all"
    min_ttl                = 0
    default_ttl            = 3600
    max_ttl                = 86400
  }
 
  restrictions {
    geo_restriction {
      restriction_type = "none"
      }
  }

  tags = {
    Environment = "production"
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}
    
resource "null_resource" "null"{
    depends_on=[
               aws_volume_attachment.attachment,
               aws_instance.web1
           ]
 
connection{
     type="ssh"
     user="ec2user"
     private_key=file("C:/Users/Dhaval/Downloads/firstkey.pem")
     host ="aws_instance.web.public_ip"
    }

 provisioner "remote-exec"{
    inline=[
         "sudo mkfs.ext4/dev/xvdh",
         "sudo mount/dev/xvdh/var/www/html",
         "sudo em -rf/var/www/html",
         "sudo git clone https://github.com/dhwanivora26/awsproject1/var/www/html/"
      ]
}
}

resource "null_resource" "null2"{
  depends_on=[
       null_resource.null,
   ]
  provisioner "local-exec"{
command="chrome ${aws_instance.web1.public_ip}"
}
}
