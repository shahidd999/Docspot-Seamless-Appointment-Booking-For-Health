import React, { useState } from 'react';
import axios from 'axios';
import { Button, Form, message } from 'antd';
import { UploadOutlined } from '@ant-design/icons';
import { Typography } from 'antd';
const AddDocs = () => {
  const [file, setFile] = useState(null);
  const [uploading, setUploading] = useState(false);

  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    if (!selectedFile) return;

    // PDF type check
    if (selectedFile.type !== 'application/pdf') {
      message.error('Only PDF files are allowed');
      setFile(null);
      e.target.value = ''; // Clear file input
      return;
    }

    setFile(selectedFile);
  };

  const handleUpload = async () => {
    if (!file) {
      message.warning("Please select a PDF file to upload");
      return;
    }

    const user = JSON.parse(localStorage.getItem('userData'));
    if (!user || !user._id) {
      message.error("User not found");
      return;
    }

    const formData = new FormData();
    formData.append('document', file);
    formData.append('userId', user._id); // Pass userId to backend

    try {
      setUploading(true);
      const res = await axios.post('http://localhost:8001/api/doctors/uploaddocument', formData, {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem("token")}`,
          'Content-Type': 'multipart/form-data'
        }
      });

      if (res.data.success) {
        message.success('Document uploaded successfully');
        setFile(null);
      } else {
        message.error(res.data.message || 'Upload failed');
      }
    } catch (error) {
      console.error("Upload error:", error);
      message.error('Something went wrong during upload');
    } finally {
      setUploading(false);
    }
  };

  return (
    <div style={{
      maxWidth: 500,
      margin: '40px auto',
      padding: '24px',
      borderRadius: '10px',
      backgroundColor: '#fff',
      boxShadow: '0 4px 12px rgba(0,0,0,0.1)'
    }}>
      <h2 style={{ textAlign: 'center', marginBottom: 20 }}>Upload Medical Document (PDF Only)</h2>
      <Form layout="vertical">
       <Form.Item label="Choose PDF File">
  <>
    <input type="file" accept="application/pdf" onChange={handleFileChange} />
    <Typography.Text type="danger" style={{ display: 'block', marginTop: 8 }}>
      ⚠ Only PDF files are allowed. Uploading other file types will be rejected.
    </Typography.Text>
  </>
</Form.Item>

        <Form.Item>
          <Button
            type="primary"
            icon={<UploadOutlined />}
            onClick={handleUpload}
            loading={uploading}
            block
            disabled={!file}
          >
            {uploading ? 'Uploading...' : 'Upload PDF'}
          </Button>
        </Form.Item>
      </Form>
    </div>
  );
};

export default AddDocs;
