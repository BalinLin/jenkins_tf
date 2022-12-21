pipeline {
    agent any

    parameters {
        string(name: 'environment', defaultValue: 'terraform', description: 'Workspace/environment file to use for deployment')
        string(name: 'bucket', defaultValue: 'balin-bucket-01', description: 'Should be manually created before running this code')
        string(name: 'key', defaultValue: 'terraformstate/jenkins-lab-01.tfstate', description: 'Where to save tfstate in S3 bucket')
        string(name: 'aws_region', defaultValue: 'ap-northeast-1', description: 'AWS region')
        string(name: 'aws_profile', defaultValue: 'balin-lab-01', description: 'Access profile')
        string(name: 'vpc_cidr', defaultValue: '10.1.0.0/16', description: 'VPC cide for deployment')
        string(name: 'vpc_subnet_newbits', defaultValue: '8', description: 'Number of bit after VPC mask')
        string(name: 'vpc_subnet_count', defaultValue: '2', description: 'Number of subnet')
        string(name: 'stack_name', defaultValue: 'balin-jenkins-lab-01', description: 'Tag name')
        string(name: 'ec2_instance_type', defaultValue: 't3.medium', description: 'Instance type')
        string(name: 'ec2_instance_count', defaultValue: '2', description: 'Number of instanc')
        string(name: 'ec2_ami', defaultValue: 'ami-072bfb8ae2c884cc4', description: 'AMI spec')
        string(name: 'zone_id', defaultValue: 'Z07739512F309N3Y0DWQ4', description: 'Route 53 host zone ID')
        string(name: 'subdomain', defaultValue: 'balin-01', description: 'Subdomain for DNS')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        booleanParam(name: 'destroy', defaultValue: false, description: 'Destroy Terraform build?')

    }


     environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }


    stages {
        stage('Plan') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }

            steps {
                sh "terraform init -input=false -backend-config='bucket=${bucket}' -backend-config='key=${key}' -backend-config='region=${aws_region}'"
                sh "terraform workspace select ${environment} || terraform workspace new ${environment}"

                sh " \
                   terraform plan -input=false -out tfplan \
                   -var='aws_region=${aws_region}' \
                   -var='aws_profile=${aws_profile}' \
                   -var='vpc_cidr=${vpc_cidr}' \
                   -var='vpc_subnet_newbits=${vpc_subnet_newbits.toInteger()}' \
                   -var='vpc_subnet_count=${vpc_subnet_count.toInteger()}' \
                   -var='stack_name=${stack_name}' \
                   -var='ec2_instance_type=${ec2_instance_type}' \
                   -var='ec2_instance_count=${ec2_instance_count.toInteger()}' \
                   -var='ec2_ami=${ec2_ami}' \
                   -var='zone_id=${zone_id}' \
                   -var='subdomain=${subdomain}' \
                   "
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
               not {
                    equals expected: true, actual: params.destroy
                }
           }


           steps {
               script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }

            steps {
                sh "terraform apply -input=false tfplan"
            }
        }

        stage('Destroy') {
            when {
                equals expected: true, actual: params.destroy
            }

        steps {
           sh " \
              terraform destroy --auto-approve \
              -var='aws_region=${aws_region}' \
              -var='aws_profile=${aws_profile}' \
              -var='vpc_cidr=${vpc_cidr}' \
              -var='vpc_subnet_newbits=${vpc_subnet_newbits.toInteger()}' \
              -var='vpc_subnet_count=${vpc_subnet_count.toInteger()}' \
              -var='stack_name=${stack_name}' \
              -var='ec2_instance_type=${ec2_instance_type}' \
              -var='ec2_instance_count=${ec2_instance_count.toInteger()}' \
              -var='ec2_ami=${ec2_ami}' \
              -var='zone_id=${zone_id}' \
              -var='subdomain=${subdomain}' \
              "
        }
    }
  }
}