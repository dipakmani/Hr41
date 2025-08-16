import pandas as pd
from faker import Faker
from datetime import datetime, timedelta
import random
import time
import os
from tqdm import tqdm
from typing import List, Dict

class HRDataGenerator:
    """Optimized HR data generator for 500K records"""
    
    def __init__(self):
        self.fake = Faker()
        self._init_reference_data()
        
    def _init_reference_data(self):
        """Initialize all reference data with synchronized IDs"""
        # Core mappings
        self.gender_map = {'Male': 1, 'Female': 2, 'Other': 3}
        self.marital_map = {'Single': 1, 'Married': 2, 'Divorced': 3}
        self.ethnicity_map = {'White': 1, 'Black': 2, 'Asian': 3, 'Hispanic': 4, 'Other': 5}
        self.employment_map = {'Full-time': 1, 'Part-time': 2, 'Contract': 3}
        self.emp_status_map = {'Active': 1, 'On Leave': 2, 'Terminated': 3}
        self.exit_reason_map = {'Resignation': 1, 'Termination': 2, 'Retirement': 3}
        self.blood_group_map = {'A+': 1, 'A-': 2, 'B+': 3, 'B-': 4, 'AB+': 5, 'AB-': 6, 'O+': 7, 'O-': 8}
        self.emp_category_map = {'Executive': 1, 'Manager': 2, 'Professional': 3, 'Technical': 4, 'Administrative': 5}
        
        # Organizational structure
        self.dept_map = {
            'Executive': 1, 'IT': 2, 'Finance': 3, 'HR': 4, 
            'Marketing': 5, 'Operations': 6, 'Sales': 7, 'R&D': 8,
            'Legal': 9, 'Customer Service': 10
        }
        
        # Education mappings
        self.education_map = {
            'High School': 1, 'Associate': 2, 'Bachelor': 3, 
            'Master': 4, 'PhD': 5
        }
        
        # Degree mappings
        self.degree_map = {
            'Computer Science': 1, 'Business Administration': 2, 
            'Electrical Engineering': 3, 'Mechanical Engineering': 4,
            'Marketing': 5, 'Finance': 6, 'Human Resources': 7,
            'Biology': 8, 'Chemistry': 9, 'Physics': 10
        }
        
        # Job titles with IDs
        self.job_title_map = {
            'CEO': 1, 'CTO': 2, 'CFO': 3, 'HR Director': 4,
            'Software Engineer': 5, 'Data Analyst': 6, 'Marketing Manager': 7,
            'Sales Executive': 8, 'Operations Manager': 9, 'Research Scientist': 10,
            'Legal Counsel': 11, 'Customer Support': 12
        }
        
        # Visa types with IDs
        self.visa_type_map = {
            'H1B': 1, 'L1': 2, 'F1': 3, 'B1': 4, 'J1': 5, 'O1': 6
        }
        
        # Countries with IDs
        self.countries = [
            ('United States', 1), ('United Kingdom', 2), ('Canada', 3),
            ('Australia', 4), ('Germany', 5), ('France', 6), ('Japan', 7),
            ('India', 8), ('Brazil', 9), ('China', 10)
        ]
        
        # States with IDs
        self.states = [
            ('California', 1), ('Texas', 2), ('New York', 3), ('Florida', 4),
            ('Illinois', 5), ('Pennsylvania', 6), ('Ohio', 7), ('Georgia', 8),
            ('North Carolina', 9), ('Michigan', 10)
        ]
        
        # Cities with IDs
        self.cities = [
            ('New York', 1), ('Los Angeles', 2), ('Chicago', 3), ('Houston', 4),
            ('Phoenix', 5), ('Philadelphia', 6), ('San Antonio', 7), ('San Diego', 8),
            ('Dallas', 9), ('San Jose', 10)
        ]
        
        # Universities with IDs
        self.universities = [
            ('Harvard University', 1), ('Stanford University', 2), ('MIT', 3),
            ('University of Oxford', 4), ('University of Cambridge', 5),
            ('ETH Zurich', 6), ('University of Toronto', 7),
            ('University of Tokyo', 8), ('National University of Singapore', 9),
            ('University of Sydney', 10)
        ]
        
        # Certifications with IDs
        self.certifications = [
            ('PMP', 1), ('CPA', 2), ('CFA', 3), ('SHRM-CP', 4),
            ('AWS Certified', 5), ('Six Sigma', 6), ('CISSP', 7),
            ('Google Cloud', 8), ('Microsoft Certified', 9)
        ]
        
        # Languages with IDs
        self.languages = [
            ('English', 1), ('Spanish', 2), ('French', 3),
            ('German', 4), ('Mandarin', 5), ('Hindi', 6),
            ('Arabic', 7), ('Portuguese', 8), ('Russian', 9)
        ]
        
        # Skills with IDs
        self.skills = [
            ('Python', 1), ('Java', 2), ('SQL', 3), ('Project Management', 4),
            ('Data Analysis', 5), ('Marketing', 6), ('Sales', 7), ('HR Policies', 8)
        ]

    def _generate_employment_dates(self, hire_date, graduation_year):
        """Generate employment-related dates with proper sequencing"""
        today = datetime.now().date()
        dates = {
            'probation_end': hire_date + timedelta(days=90),
            'next_review': hire_date + timedelta(days=365),
            'exit_date': None,
            'last_promotion': None,
            'certification1_date': None,
            'certification2_date': None
        }
        
        if random.random() > 0.7 and hire_date < today:
            end_date = min(today, dates['probation_end'])
            if hire_date < end_date:
                dates['last_promotion'] = self.fake.date_between(
                    start_date=hire_date, 
                    end_date=end_date
                )
        
        grad_date = datetime(graduation_year, 6, 1).date()
        min_cert_date = max(
            grad_date, 
            hire_date - timedelta(days=365*2)
        )
        
        if random.random() > 0.3 and min_cert_date <= today:
            dates['certification1_date'] = self.fake.date_between(
                start_date=min_cert_date,
                end_date=today
            )
        
        if random.random() > 0.7 and min_cert_date <= today:
            dates['certification2_date'] = self.fake.date_between(
                start_date=min_cert_date,
                end_date=today
            )
        
        return dates
    
    def _get_valid_manager(self, emp_id: int, existing_employees: List[Dict]) -> Dict:
        """Get a valid manager ensuring proper hierarchy"""
        if emp_id == 1:  # CEO has no manager
            return None
        
        eligible_managers = [
            m for m in existing_employees 
            if m['EmployeeID'] != emp_id and
            m['EmploymentStatus'] == 'Active' and
            m['EmployeeID'] < emp_id
        ]
        
        if not eligible_managers:
            ceo = next((e for e in existing_employees if e['EmployeeID'] == 1), None)
            return ceo if ceo and ceo['EmploymentStatus'] == 'Active' else None
        
        return random.choice(eligible_managers)
    
    def _get_random_skills(self):
        """Get random skills with synchronized IDs"""
        selected = random.sample(self.skills, min(3, len(self.skills)))
        skill_names = [s[0] for s in selected]
        skill_ids = [str(s[1]) for s in selected]
        return ', '.join(skill_names), ', '.join(skill_ids)
    
    def _get_random_job_title(self):
        """Get random job title with synchronized ID"""
        title = random.choice(list(self.job_title_map.keys()))
        return title, self.job_title_map[title]
    
    def _get_random_visa_type(self):
        """Get random visa type with synchronized ID or None"""
        if random.random() < 0.3:
            visa_type = random.choice(list(self.visa_type_map.keys()))
            return visa_type, self.visa_type_map[visa_type]
        return None, None
    
    def _generate_employee(self, emp_id: int, existing_employees: List[Dict]) -> Dict:
        """Generate a single employee record"""
        manager = self._get_valid_manager(emp_id, existing_employees)
        today = datetime.now().date()
        
        # Generate synchronized values
        emp_type, emp_type_id = random.choice(list(self.employment_map.items()))
        dept, dept_id = random.choice(list(self.dept_map.items()))
        job_title, job_title_id = self._get_random_job_title()
        country, country_id = random.choice(self.countries)
        state, state_id = random.choice(self.states)
        city, city_id = random.choice(self.cities)
        university, university_id = random.choice(self.universities)
        degree, degree_id = random.choice(list(self.degree_map.items()))
        skills, skill_ids = self._get_random_skills()
        visa_type, visa_type_id = self._get_random_visa_type()
        
        # Generate dates
        hire_date = self.fake.date_between(start_date='-10y', end_date='-6m')
        graduation_year = random.randint(max(1990, hire_date.year - 30), hire_date.year - 1)
        dates = self._generate_employment_dates(hire_date, graduation_year)
        
        # Determine employment status
        if random.random() < 0.15:
            probation_end = hire_date + timedelta(days=90)
            if probation_end <= today:
                dates['exit_date'] = self.fake.date_between(
                    start_date=probation_end,
                    end_date=today
                )
                emp_status = 'Terminated'
                exit_reason, exit_reason_id = random.choice(list(self.exit_reason_map.items()))
            else:
                emp_status = 'Active'
                exit_reason, exit_reason_id = None, None
        else:
            emp_status = 'Active'
            exit_reason, exit_reason_id = None, None
        
        # Certifications
        cert1, cert1_id = random.choice(self.certifications)
        cert2, cert2_id = random.choice([c for c in self.certifications if c[0] != cert1] + [(None, None)])

        # Languages
        lang1, lang1_id = random.choice(self.languages)
        lang2, lang2_id = random.choice([l for l in self.languages if l[0] != lang1] + [(None, None)])

        # Build record
        record = {
            'EmployeeID': emp_id,
            'FirstName': self.fake.first_name(),
            'LastName': self.fake.last_name(),
            'DateOfBirth': self.fake.date_of_birth(minimum_age=22, maximum_age=65),
            'Gender': (gender := random.choice(list(self.gender_map.keys()))),
            'GenderID': self.gender_map[gender],
            'Nationality': country,
            'NationalityID': country_id,
            'MaritalStatus': (status := random.choice(list(self.marital_map.keys()))),
            'MaritalStatusID': self.marital_map[status],
            'ContactNumber': self.fake.phone_number(),
            'Email': None,  # Will be set later
            'AddressLine1': self.fake.street_address(),
            'AddressLine2': self.fake.secondary_address(),
            'City': city,
            'CityID': city_id,
            'State': state,
            'StateID': state_id,
            'PostalCode': self.fake.postcode(),
            'Country': country,
            'CountryID': country_id,
            'JoiningDate': hire_date,
            'EmploymentType': emp_type,
            'EmploymentTypeID': emp_type_id,
            'DepartmentID': dept_id,
            'DepartmentName': dept,
            'JobTitle': job_title,
            'JobTitleID': job_title_id,
            'JobLevel': random.randint(1, 10),
            'ManagerID': manager['EmployeeID'] if manager else None,
            'ManagerName': f"{manager['FirstName']} {manager['LastName']}" if manager else None,
            'CostCenter': f"CC{random.randint(1000, 9999)}",
            'CostCenterID': random.randint(1, 100),
            'LocationCode': f"LOC{random.randint(1, 20)}",
            'LocationID': random.randint(1, 20),
            'EmploymentStatus': emp_status,
            'EmploymentStatusID': self.emp_status_map[emp_status],
            'ExitDate': dates['exit_date'],
            'ExitReason': exit_reason,
            'ExitReasonID': exit_reason_id,
            'WorkEmail': None,
            'WorkPhone': self.fake.phone_number(),
            'Extension': str(random.randint(1000, 9999)),
            'BadgeID': f"BG{random.randint(10000, 99999)}",
            'IsActive': emp_status == 'Active',
            'ProbationEndDate': dates['probation_end'],
            'PayGrade': f"PG{random.randint(1, 10)}",
            'PayGradeID': random.randint(1, 10),
            'EmployeeCategory': (category := random.choice(list(self.emp_category_map.keys()))),
            'EmployeeCategoryID': self.emp_category_map[category],
            'UnionMembership': random.choice([True, False]),
            'UnionID': random.randint(1, 5) if random.choice([True, False]) else None,
            'Ethnicity': (ethnicity := random.choice(list(self.ethnicity_map.keys()))),
            'EthnicityID': self.ethnicity_map[ethnicity],
            'DisabilityStatus': random.choice([True, False]),
            'DisabilityID': random.randint(1, 10) if random.choice([True, False]) else None,
            'VeteranStatus': random.choice([True, False]),
            'VeteranID': random.randint(1, 5) if random.choice([True, False]) else None,
            'EmergencyContactName': self.fake.name(),
            'EmergencyContactPhone': self.fake.phone_number(),
            'EmergencyContactRelation': random.choice(['Spouse', 'Parent', 'Sibling', 'Friend']),
            'EmergencyContactID': random.randint(1, 100),
            'BloodGroup': (blood := random.choice(list(self.blood_group_map.keys()))),
            'BloodGroupID': self.blood_group_map[blood],
            'PassportNumber': self.fake.passport_number() if random.random() < 0.7 else None,
            'PassportExpiry': self.fake.future_date(end_date='+10y') if random.random() < 0.7 else None,
            'VisaType': visa_type,
            'VisaTypeID': visa_type_id,
            'VisaExpiry': self.fake.future_date(end_date='+3y') if visa_type else None,
            'DrivingLicenseNumber': self.fake.license_plate() if random.random() < 0.8 else None,
            'DrivingLicenseExpiry': self.fake.future_date(end_date='+5y') if random.random() < 0.8 else None,
            'EducationLevel': (edu := random.choice(list(self.education_map.keys()))),
            'EducationLevelID': self.education_map[edu],
            'University': university,
            'UniversityID': university_id,
            'Degree': degree,
            'DegreeID': degree_id,
            'GraduationYear': graduation_year,
            'Certification1': cert1,
            'Certification1ID': cert1_id,
            'Certification1Date': dates['certification1_date'],
            'Certification2': cert2,
            'Certification2ID': cert2_id,
            'Certification2Date': dates['certification2_date'],
            'Skills': skills,
            'SkillIDs': skill_ids,
            'Language1': lang1,
            'Language1ID': lang1_id,
            'Language2': lang2,
            'Language2ID': lang2_id,
            'LinkedInProfile': f"linkedin.com/in/{self.fake.user_name()}",
            'EmployeePhoto': f"photos/employee_{emp_id}.jpg",
            'LastPromotionDate': dates['last_promotion'],
            'NextReviewDate': dates['next_review'],
            'WorkSchedule': random.choice(['9-5', 'Flexible', 'Shift', 'Remote']),
            'WorkScheduleID': random.randint(1, 4),
            'RemoteWorkEligible': random.choice([True, False]),
            'SocialSecurityNumber': self.fake.ssn() if random.choice([True, False]) else None,
            'BankName': self.fake.company() + " Bank",
            'BankAccountNumber': self.fake.bban()
        }
        
        # Set emails after name is generated
        record['Email'] = f"{record['FirstName'][0].lower()}{record['LastName'].lower()}@company.com"
        record['WorkEmail'] = f"{record['FirstName'][0].lower()}{record['LastName'].lower()}@work.company.com"
        
        return record

    def _generate_employee_chunk(self, start_id: int, count: int) -> List[Dict]:
        """Generate a chunk of employee records"""
        chunk = []
        existing_employees = [] if start_id == 1 else [{'EmployeeID': 1, 'FirstName': 'Robert', 'LastName': 'Johnson', 'EmploymentStatus': 'Active'}]
        
        for emp_id in range(start_id, start_id + count):
            if emp_id == 1:  # CEO record
                chunk.append(self._generate_ceo())
                continue
                
            record = self._generate_employee(emp_id, existing_employees)
            chunk.append(record)
            existing_employees.append(record)
            
        return chunk

    def _generate_ceo(self) -> Dict:
        """Generate the CEO record"""
        return {
            'EmployeeID': 1,
            'FirstName': 'Robert',
            'LastName': 'Johnson',
            'DateOfBirth': datetime(1965, 1, 1).date(),
            'Gender': 'Male',
            'GenderID': 1,
            'Nationality': 'United States',
            'NationalityID': 1,
            'MaritalStatus': 'Married',
            'MaritalStatusID': 2,
            'ContactNumber': '+1 555-010-0001',
            'Email': 'rjohnson@company.com',
            'AddressLine1': '1 Executive Blvd',
            'AddressLine2': 'Suite 1000',
            'City': 'New York',
            'CityID': 1,
            'State': 'New York',
            'StateID': 3,
            'PostalCode': '10001',
            'Country': 'United States',
            'CountryID': 1,
            'JoiningDate': datetime(2010, 1, 1).date(),
            'EmploymentType': 'Full-time',
            'EmploymentTypeID': 1,
            'DepartmentID': 1,
            'DepartmentName': 'Executive',
            'JobTitle': 'CEO',
            'JobTitleID': 1,
            'JobLevel': 10,
            'ManagerID': None,
            'ManagerName': None,
            'CostCenter': 'CC1000',
            'CostCenterID': 1,
            'LocationCode': 'LOC1',
            'LocationID': 1,
            'EmploymentStatus': 'Active',
            'EmploymentStatusID': 1,
            'ExitDate': None,
            'ExitReason': None,
            'ExitReasonID': None,
            'WorkEmail': 'rjohnson@work.company.com',
            'WorkPhone': '+1 555-010-0002',
            'Extension': '1000',
            'BadgeID': 'BG10000',
            'IsActive': True,
            'ProbationEndDate': None,
            'PayGrade': 'PG10',
            'PayGradeID': 10,
            'EmployeeCategory': 'Executive',
            'EmployeeCategoryID': 1,
            'UnionMembership': False,
            'UnionID': None,
            'Ethnicity': 'White',
            'EthnicityID': 1,
            'DisabilityStatus': False,
            'DisabilityID': None,
            'VeteranStatus': True,
            'VeteranID': 1,
            'EmergencyContactName': 'Sarah Johnson',
            'EmergencyContactPhone': '+1 555-010-0003',
            'EmergencyContactRelation': 'Spouse',
            'EmergencyContactID': 1,
            'BloodGroup': 'A+',
            'BloodGroupID': 1,
            'PassportNumber': 'P12345678',
            'PassportExpiry': datetime(2030, 1, 1).date(),
            'VisaType': None,
            'VisaTypeID': None,
            'VisaExpiry': None,
            'DrivingLicenseNumber': 'DL1000001',
            'DrivingLicenseExpiry': datetime(2028, 1, 1).date(),
            'EducationLevel': 'PhD',
            'EducationLevelID': 5,
            'University': 'Harvard University',
            'UniversityID': 1,
            'Degree': 'Business Administration',
            'DegreeID': 2,
            'GraduationYear': 1990,
            'Certification1': 'PMP',
            'Certification1ID': 1,
            'Certification1Date': datetime(2000, 1, 1).date(),
            'Certification2': 'CFA',
            'Certification2ID': 3,
            'Certification2Date': datetime(2005, 1, 1).date(),
            'Skills': 'Leadership, Strategy, Business Development',
            'SkillIDs': '101, 102, 103',
            'Language1': 'English',
            'Language1ID': 1,
            'Language2': 'French',
            'Language2ID': 3,
            'LinkedInProfile': 'linkedin.com/in/robertjohnson',
            'EmployeePhoto': 'photos/employee_1.jpg',
            'LastPromotionDate': None,
            'NextReviewDate': datetime.now().date() + timedelta(days=365),
            'WorkSchedule': 'Flexible',
            'WorkScheduleID': 2,
            'RemoteWorkEligible': True,
            'SocialSecurityNumber': '123-45-6789',
            'BankName': 'Global Trust Bank',
            'BankAccountNumber': '1234567890'
        }

    def generate_sample(self, total_records: int = 500000, chunk_size: int = 50000) -> None:
        """Generate and save data in optimized chunks"""
        print(f"Generating {total_records:,} HR records...")
        start_time = time.time()
        
        # Prepare output file
        output_file = 'hr_records_500k.csv'
        sample_record = self._generate_ceo()
        pd.DataFrame([sample_record]).head(0).to_csv(output_file, index=False)
        
        # Process in chunks with progress bar
        chunks = range(1, total_records + 1, chunk_size)
        for chunk_start in tqdm(chunks, desc='Generating records'):
            chunk = self._generate_employee_chunk(
                start_id=chunk_start,
                count=min(chunk_size, total_records - chunk_start + 1)
            )
            
            pd.DataFrame(chunk).to_csv(
                output_file,
                mode='a',
                header=False,
                index=False
            )
        
        file_size = os.path.getsize(output_file)/(1024**2)
        print(f"\nSuccess! Generated {total_records:,} records in {(time.time()-start_time)/60:.1f} minutes")
        print(f"File saved to: {output_file} ({file_size:.1f} MB)")

# Main execution
if __name__ == "__main__":
    generator = HRDataGenerator()
    generator.generate_sample(total_records=500000)
