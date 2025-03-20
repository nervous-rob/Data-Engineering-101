# Lesson 6.4: Data Security Fundamentals

## Navigation
- [← Back to Lesson Plan](../6.4-data-security-fundamentals.md)
- [← Back to Module Overview](../README.md)

## Overview
This lesson explores fundamental concepts and practical implementations of data security in modern computing environments. We'll cover essential security principles, encryption techniques, access control mechanisms, and best practices for protecting sensitive data. The content incorporates the latest security standards, including NIST's post-quantum cryptography guidelines and zero trust architecture principles.

## Learning Objectives
After completing this lesson, you'll be able to:
- Implement comprehensive data security solutions using modern standards
- Apply encryption techniques effectively, including post-quantum cryptography
- Design and implement robust access control systems
- Deploy security monitoring and audit mechanisms
- Implement zero trust architecture principles
- Maintain compliance with security standards and regulations

## Key Topics

### 1. Modern Data Security Implementation

#### Encryption Service Implementation

```python
from typing import Dict, List, Optional, Any
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
import base64
import logging
import os

class DataEncryptionService:
    """Implements modern data encryption patterns"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_encryption()
        
    def _initialize_encryption(self) -> None:
        """Initialize encryption components"""
        try:
            # Generate or load master key
            self.master_key = self._get_master_key()
            
            # Initialize key derivation function
            self.kdf = PBKDF2HMAC(
                algorithm=hashes.SHA256(),
                length=32,
                salt=self.config.get('salt', os.urandom(16)),
                iterations=100000
            )
            
            # Initialize Fernet for symmetric encryption
            self.fernet = Fernet(base64.urlsafe_b64encode(self.master_key))
            
        except Exception as e:
            self.logger.error(f"Failed to initialize encryption: {str(e)}")
            raise
            
    def _get_master_key(self) -> bytes:
        """Get or generate master encryption key"""
        try:
            if 'master_key_path' in self.config:
                return self._load_master_key(self.config['master_key_path'])
            else:
                return self._generate_master_key()
                
        except Exception as e:
            self.logger.error(f"Failed to get master key: {str(e)}")
            raise
            
    async def encrypt_data(
        self,
        data: bytes,
        context: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Encrypt data with context-aware security"""
        try:
            # Generate data key
            data_key = os.urandom(32)
            
            # Encrypt data key with master key
            encrypted_data_key = self.fernet.encrypt(data_key)
            
            # Create encryption context
            encryption_context = {
                'algorithm': 'AES-256-GCM',
                'key_id': self.config['key_id'],
                'timestamp': context.get('timestamp'),
                'purpose': context.get('purpose', 'general')
            }
            
            # Encrypt data with data key
            encrypted_data = self._encrypt_with_context(
                data,
                data_key,
                encryption_context
            )
            
            return {
                'encrypted_data': encrypted_data,
                'encrypted_data_key': encrypted_data_key,
                'encryption_context': encryption_context
            }
            
        except Exception as e:
            self.logger.error(f"Failed to encrypt data: {str(e)}")
            raise
            
    async def decrypt_data(
        self,
        encrypted_package: Dict[str, Any]
    ) -> bytes:
        """Decrypt data with security validation"""
        try:
            # Decrypt data key
            data_key = self.fernet.decrypt(
                encrypted_package['encrypted_data_key']
            )
            
            # Validate encryption context
            if not self._validate_encryption_context(
                encrypted_package['encryption_context']
            ):
                raise ValueError("Invalid encryption context")
                
            # Decrypt data
            decrypted_data = self._decrypt_with_context(
                encrypted_package['encrypted_data'],
                data_key,
                encrypted_package['encryption_context']
            )
            
            return decrypted_data
            
        except Exception as e:
            self.logger.error(f"Failed to decrypt data: {str(e)}")
            raise
```

#### Access Control Implementation

```python
from typing import Dict, List, Optional, Any
import jwt
from datetime import datetime, timedelta
import logging

class AccessControlService:
    """Implements modern access control patterns"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_service()
        
    def _initialize_service(self) -> None:
        """Initialize access control components"""
        try:
            # Load security policies
            self.policies = self._load_security_policies()
            
            # Initialize token service
            self.token_service = self._initialize_token_service()
            
            # Set up authentication providers
            self.auth_providers = self._setup_auth_providers()
            
        except Exception as e:
            self.logger.error(f"Failed to initialize access control: {str(e)}")
            raise
            
    async def authenticate_user(
        self,
        credentials: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Authenticate user with multi-factor support"""
        try:
            # Primary authentication
            auth_result = await self._primary_authentication(credentials)
            
            # MFA if required
            if self._requires_mfa(auth_result['user'], auth_result['context']):
                mfa_result = await self._verify_mfa(
                    auth_result['user'],
                    credentials.get('mfa_token')
                )
                auth_result['mfa_verified'] = mfa_result
                
            # Generate session token
            token = self._generate_session_token(auth_result)
            
            return {
                'token': token,
                'user_info': auth_result['user'],
                'permissions': auth_result['permissions']
            }
            
        except Exception as e:
            self.logger.error(f"Authentication failed: {str(e)}")
            raise
            
    async def authorize_access(
        self,
        token: str,
        resource: str,
        action: str
    ) -> bool:
        """Authorize access with zero trust principles"""
        try:
            # Validate token
            token_data = self._validate_token(token)
            
            # Check resource permissions
            if not self._check_resource_permissions(
                token_data['user'],
                resource,
                action
            ):
                return False
                
            # Verify context
            if not self._verify_access_context(
                token_data['context'],
                resource
            ):
                return False
                
            # Log access attempt
            await self._log_access_attempt(
                token_data,
                resource,
                action,
                True
            )
            
            return True
            
        except Exception as e:
            self.logger.error(f"Authorization failed: {str(e)}")
            await self._log_access_attempt(
                token_data if 'token_data' in locals() else None,
                resource,
                action,
                False
            )
            return False 
```

### 2. Post-Quantum Cryptography Implementation

Based on NIST's finalized standards (FIPS 203, 204, and 205), here's an implementation of post-quantum cryptography:

```python
from typing import Dict, List, Optional, Any
import logging
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import padding
import os

class PostQuantumCrypto:
    """Implements NIST-standardized post-quantum cryptography"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_pqc()
        
    def _initialize_pqc(self) -> None:
        """Initialize post-quantum cryptography components"""
        try:
            # Initialize ML-KEM (CRYSTALS-Kyber) for general encryption
            self.ml_kem = self._setup_ml_kem()
            
            # Initialize ML-DSA (CRYSTALS-Dilithium) for digital signatures
            self.ml_dsa = self._setup_ml_dsa()
            
            # Initialize SLH-DSA (Sphincs+) as backup signature scheme
            self.slh_dsa = self._setup_slh_dsa()
            
        except Exception as e:
            self.logger.error(f"Failed to initialize PQC: {str(e)}")
            raise
            
    async def generate_keys(self) -> Dict[str, Any]:
        """Generate post-quantum secure key pairs"""
        try:
            # Generate ML-KEM key pair
            kem_public_key, kem_private_key = self.ml_kem.keygen()
            
            # Generate ML-DSA key pair
            dsa_public_key, dsa_private_key = self.ml_dsa.keygen()
            
            return {
                'encryption': {
                    'public_key': kem_public_key,
                    'private_key': kem_private_key
                },
                'signature': {
                    'public_key': dsa_public_key,
                    'private_key': dsa_private_key
                }
            }
            
        except Exception as e:
            self.logger.error(f"Failed to generate PQ keys: {str(e)}")
            raise
            
    async def encrypt_message(
        self,
        message: bytes,
        recipient_public_key: bytes
    ) -> Dict[str, Any]:
        """Encrypt message using ML-KEM"""
        try:
            # Generate shared secret and ciphertext
            shared_secret, ciphertext = self.ml_kem.encapsulate(
                recipient_public_key
            )
            
            # Encrypt message with shared secret
            encrypted_message = self._encrypt_with_shared_secret(
                message,
                shared_secret
            )
            
            return {
                'ciphertext': ciphertext,
                'encrypted_message': encrypted_message
            }
            
        except Exception as e:
            self.logger.error(f"Failed to encrypt with PQC: {str(e)}")
            raise
```

### 3. Zero Trust Architecture Implementation

Following NIST's latest guidance on Zero Trust Architecture (ZTA):

```python
from typing import Dict, List, Optional, Any
import logging
from datetime import datetime
import jwt

class ZeroTrustController:
    """Implements NIST Zero Trust Architecture principles"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_zta()
        
    def _initialize_zta(self) -> None:
        """Initialize Zero Trust components"""
        try:
            # Initialize Policy Engine
            self.policy_engine = self._setup_policy_engine()
            
            # Initialize Policy Administrator
            self.policy_admin = self._setup_policy_admin()
            
            # Initialize Trust Algorithm
            self.trust_algorithm = self._setup_trust_algorithm()
            
        except Exception as e:
            self.logger.error(f"Failed to initialize ZTA: {str(e)}")
            raise
            
    async def evaluate_access_request(
        self,
        subject: Dict[str, Any],
        resource: Dict[str, Any],
        context: Dict[str, Any]
    ) -> Dict[str, Any]:
        """Evaluate access request using Zero Trust principles"""
        try:
            # Collect trust evidence
            trust_evidence = await self._collect_trust_evidence(
                subject,
                resource,
                context
            )
            
            # Calculate trust score
            trust_score = self.trust_algorithm.calculate_score(
                trust_evidence
            )
            
            # Evaluate policies
            policy_decision = await self.policy_engine.evaluate(
                subject,
                resource,
                context,
                trust_score
            )
            
            # Generate access token if approved
            if policy_decision['approved']:
                access_token = self._generate_limited_token(
                    subject,
                    resource,
                    policy_decision['constraints']
                )
                policy_decision['access_token'] = access_token
                
            return policy_decision
            
        except Exception as e:
            self.logger.error(f"Failed to evaluate ZTA request: {str(e)}")
            raise
```

### 4. Security Monitoring and Audit Implementation

```python
from typing import Dict, List, Optional, Any
import logging
from datetime import datetime
import json
import asyncio

class SecurityMonitor:
    """Implements comprehensive security monitoring"""
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self._initialize_monitoring()
        
    def _initialize_monitoring(self) -> None:
        """Initialize monitoring components"""
        try:
            # Initialize event collectors
            self.collectors = self._setup_collectors()
            
            # Initialize alert system
            self.alert_system = self._setup_alert_system()
            
            # Initialize audit log
            self.audit_log = self._setup_audit_log()
            
        except Exception as e:
            self.logger.error(f"Failed to initialize monitoring: {str(e)}")
            raise
            
    async def monitor_security_events(self) -> None:
        """Monitor security events in real-time"""
        try:
            while True:
                # Collect security events
                events = await self._collect_security_events()
                
                # Analyze events
                alerts = self._analyze_events(events)
                
                # Process alerts
                if alerts:
                    await self._process_alerts(alerts)
                    
                # Update audit log
                await self._update_audit_log(events)
                
                # Brief delay before next collection
                await asyncio.sleep(self.config['collection_interval'])
                
        except Exception as e:
            self.logger.error(f"Security monitoring failed: {str(e)}")
            raise
            
    async def generate_security_report(
        self,
        start_time: datetime,
        end_time: datetime
    ) -> Dict[str, Any]:
        """Generate security report for specified time period"""
        try:
            # Collect audit logs
            audit_entries = await self._get_audit_entries(
                start_time,
                end_time
            )
            
            # Analyze security metrics
            metrics = self._calculate_security_metrics(audit_entries)
            
            # Generate insights
            insights = self._generate_security_insights(
                metrics,
                audit_entries
            )
            
            return {
                'time_period': {
                    'start': start_time,
                    'end': end_time
                },
                'metrics': metrics,
                'insights': insights,
                'recommendations': self._generate_recommendations(insights)
            }
            
        except Exception as e:
            self.logger.error(f"Failed to generate security report: {str(e)}")
            raise
```

## Best Practices and Guidelines

1. **Data Protection**
   - Always encrypt data at rest and in transit
   - Implement post-quantum cryptography for future-proof security
   - Use key rotation and secure key management
   - Apply the principle of least privilege

2. **Access Control**
   - Implement multi-factor authentication
   - Use role-based access control (RBAC)
   - Regularly audit access permissions
   - Implement session management and timeout

3. **Zero Trust Architecture**
   - Never trust, always verify
   - Implement micro-segmentation
   - Use continuous monitoring and validation
   - Apply adaptive policies based on risk

4. **Monitoring and Audit**
   - Implement comprehensive logging
   - Set up real-time alerting
   - Conduct regular security assessments
   - Maintain audit trails for compliance

## Practice Exercises

1. Implement a post-quantum key exchange using ML-KEM
2. Design a Zero Trust access control system
3. Create a security monitoring dashboard
4. Implement an audit logging system

## Review Questions

1. What are the key components of NIST's post-quantum cryptography standards?
2. How does Zero Trust Architecture differ from traditional security models?
3. What are the essential elements of a comprehensive security monitoring system?
4. How do you implement least privilege access in a modern application?

## Additional Resources

- [NIST Post-Quantum Cryptography Standards](https://www.nist.gov/pqcrypto)
- [Zero Trust Architecture Guidelines](https://www.nist.gov/zerotrust)
- [Security Monitoring Best Practices](https://www.nist.gov/cyberframework)
- [Data Protection Guidelines](https://www.nist.gov/privacy-framework)

## Next Steps

- Review the provided code implementations
- Complete the practice exercises
- Explore the additional resources
- Prepare for the hands-on lab session 