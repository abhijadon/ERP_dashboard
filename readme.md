import { useEffect, useState } from 'react';
import Card from '@mui/joy/Card';
import CardContent from '@mui/joy/CardContent';
import Typography from '@mui/joy/Typography';
import { Progress, Select } from 'antd';
import CircularProgress from '@mui/joy/CircularProgress';
import { GrPowerReset } from "react-icons/gr";
import { AiOutlineExport } from "react-icons/ai";
import SvgIcon from '@mui/joy/SvgIcon';
import {
  EyeOutlined,
  EditOutlined,
  DeleteOutlined,
  FilePdfOutlined,
  PlusOutlined,
  EllipsisOutlined,
} from '@ant-design/icons';
import { Descriptions, Dropdown, Table, Button, Input, message } from 'antd';
import { PageHeader } from '@ant-design/pro-layout';

import { useSelector, useDispatch } from 'react-redux';
import useLanguage from '@/locale/useLanguage';
import { erp } from '@/redux/erp/actions';
import { selectListItems } from '@/redux/erp/selectors';
import { useErpContext } from '@/context/erp';
import { generate as uniqueId } from 'shortid';
import { useNavigate } from 'react-router-dom';
import useResponsiveTable from '@/hooks/useResponsiveTable';
import { DOWNLOAD_BASE_URL } from '@/config/serverApiConfig';
const { Search } = Input;




function AddNewItem({ config, hasCreate = true }) {
  const navigate = useNavigate();
  const { ADD_NEW_ENTITY, entity } = config;

  const handleClick = () => {
    navigate(`/${entity.toLowerCase()}/create`);
  };

  if (hasCreate)
    return (
      <Button onClick={handleClick} type="primary" icon={<PlusOutlined />}>
        {ADD_NEW_ENTITY}
      </Button>
    );
  else return null;
}
export default function DataTable({ config, extra = [] }) {
  const translate = useLanguage();
  let { entity, dataTableColumns, create = true } = config;
  const { result: listResult, isLoading: listIsLoading } = useSelector(selectListItems);
  const { pagination, items: dataSource } = listResult;
  const { erpContextAction } = useErpContext();
  const { modal } = erpContextAction;


  {/* filters code  */ }

  const [institutes, setInstitutes] = useState([]);
  const [universities, setUniversities] = useState([]);
  const [counselors, setCounselors] = useState([]);
  const [paymentType, setpaymentType] = useState([]);
  const [selectedInstitute, setSelectedInstitute] = useState('');
  const [selectedUniversity, setSelectedUniversity] = useState('');
  const [selectedCounselor, setSelectedCounselor] = useState('');
  const [selectedPayment, setSelectedPayment] = useState('');
  const [selectedDate, setSelectedDate] = useState('');
  const [filteredPaymentData, setFilteredPaymentData] = useState({});
  const [universityExistenceMessage, setUniversityExistenceMessage] = useState('');

  const handleInstituteChange = (value) => {
    setSelectedInstitute(value);
  };

  const handleUniversityChange = async (value) => {
    setSelectedUniversity(value);
    setUniversityExistenceMessage(''); // Reset the university existence message

    // Check if the selected university exists in the dataset
    if (value && universities.indexOf(value) === -1) {
      setUniversityExistenceMessage(`The specified university (${value}) does not exist in the dataset.`);
      setFilteredPaymentData({ total_course_fee: 0 }); // Set payment to 0
    } else {
      setFilteredPaymentData({}); // Clear filtered data
    }
  };

  const handleCounselorChange = (value) => {
    setSelectedCounselor(value);
  };
  const handlePaymentChange = (value) => {
    setSelectedPayment(value);
  };

  const handleDateChange = (e) => {
    setSelectedDate(e.target.value);
  };

  const resetData = () => {
    setSelectedInstitute('');
    setSelectedUniversity('');
    setSelectedCounselor('');
    setSelectedPayment('');
    setSelectedDate('');
    setFilteredPaymentData({});
    setUniversityExistenceMessage('');
  };
  const fetchData = async () => {
    try {
      const response = await fetch(`${import.meta.env.VITE_BACKEND_SERVER}api/payment/filter`);
      const data = await response.json();

      if (data.success && data.result !== null) {
        const uniqueInstitutes = Array.isArray(data.result) ? [...new Set(data.result.map((item) => item.institute_name))] : [];
        const uniqueUniversities = Array.isArray(data.result) ? [...new Set(data.result.map((item) => item.university_name))] : [];
        const uniqueCounselors = Array.isArray(data.result) ? [...new Set(data.result.map((item) => item.counselor_email))] : [];
        const uniquePayment = Array.isArray(data.result) ? [...new Set(data.result.map((item) => item.payment_type))] : [];

        setInstitutes(uniqueInstitutes);
        setUniversities(uniqueUniversities);
        setCounselors(uniqueCounselors);
        setpaymentType(uniquePayment);
      }
    } catch (error) {
      console.error('Error fetching data:', error);
    }
  };

  useEffect(() => {
    fetchData();
  }, []);


  {/* filters code  */ }


  useEffect(() => {
    const controller = new AbortController();

    const fetchData = async () => {
      if (selectedInstitute || selectedUniversity || selectedCounselor || selectedDate || selectedPayment) {
        try {
          const response = await fetch(
            `${import.meta.env.VITE_BACKEND_SERVER}api/payment/summary?institute_name=${selectedInstitute}&university_name=${selectedUniversity}&counselor_email=${selectedCounselor}&date=${selectedDate}&payment_type=${selectedPayment}`
          );
          const data = await response.json();
          console.log('Abhishek jadon', data)
          // After setting filteredPaymentData
          console.log('Filtered Payment Data:', filteredPaymentData);
          if (data.success && data.result !== null) {
            setFilteredPaymentData(data.result || {});

            let successMessage = 'Data fetched successfully.';
            if (selectedUniversity) {
              successMessage = `Data fetched successfully for the specified university: ${selectedUniversity}.`;
            } else if (selectedInstitute) {
              successMessage = `Data fetched successfully for the specified institute: ${selectedInstitute}.`;
            } else if (selectedCounselor) {
              successMessage = `Data fetched successfully for the specified counselor: ${getEmailName(selectedCounselor)}.`;
            } else if (selectedDate) {
              successMessage = `Data fetched successfully for the specified date: ${selectedDate}.`;
            }
            else if (selectedPayment) {
              successMessage = `Data fetched successfully for the specified date: ${selectedPayment}.`;
            }
            message.success(successMessage);
          } else {
            setFilteredPaymentData({ total_course_fee: 0 }); // Set payment to 0
            let errorMessage = 'No data found based on the specified filters.';
            if (selectedUniversity) {
              errorMessage = `No data found for the specified filters and university: ${selectedUniversity}.`;
            } else if (selectedInstitute) {
              errorMessage = `No data found for the specified filters and institute: ${selectedInstitute}.`;
            } else if (selectedCounselor) {
              errorMessage = `No data found for the specified filters and counselor: ${getEmailName(selectedCounselor)}.`;
            } else if (selectedDate) {
              errorMessage = `No data found for the specified filters and date: ${selectedDate}.`;
            }
            else if (selectedPayment) {
              errorMessage = `No data found for the specified filters and date: ${selectedPayment}.`;
            }
            message.error(errorMessage);
          }
        } catch (error) {
          console.error('Error fetching data:', error);
        }
      }
    };

    const getEmailName = (email) => {
      console.log('Email:', email); // Log the email value
      if (!email) return '';
      const parts = email.split('@');
      return parts[0];
    };
    fetchData();

    return () => {
      controller.abort();
    };
  }, [selectedInstitute, selectedUniversity, selectedCounselor, selectedDate, selectedPayment]);


  const items = [
    {
      label: translate('Show'),
      key: 'read',
      icon: <EyeOutlined />,
    },
    {
      label: translate('Edit'),
      key: 'edit',
      icon: <EditOutlined />,
    },
    {
      label: translate('Download'),
      key: 'download',
      icon: <FilePdfOutlined />,
    },
    ...extra,
    {
      type: 'divider',
    },

    {
      label: translate('Delete'),
      key: 'delete',
      icon: <DeleteOutlined />,
    },
  ];

  const navigate = useNavigate();

  const handleRead = (record) => {
    dispatch(erp.currentItem({ data: record }));
    navigate(`/${entity}/read/${record._id}`);
  };
  const handleEdit = (record) => {
    dispatch(erp.currentAction({ actionType: 'update', data: record }));
    navigate(`/${entity}/update/${record._id}`);
  };
  const handleDownload = (record) => {
    window.open(`${DOWNLOAD_BASE_URL}${entity}/${entity}-${record._id}.pdf`, '_blank');
  };

  const handleDelete = (record) => {
    dispatch(erp.currentAction({ actionType: 'delete', data: record }));
    modal.open();
  };

  const handleRecordPayment = (record) => {
    dispatch(erp.currentItem({ data: record }));
    navigate(`/invoice/pay/${record._id}`);
  };

  dataTableColumns = [
    ...dataTableColumns,
    {
      title: 'Action',
      key: 'action',
      fixed: 'right',
      render: (_, record) => (
        <Dropdown
          menu={{
            items,
            onClick: ({ key }) => {
              switch (key) {
                case 'read':
                  handleRead(record);
                  break;
                case 'edit':
                  handleEdit(record);
                  break;
                case 'download':
                  handleDownload(record);
                  break;
                case 'delete':
                  handleDelete(record);
                  break;
                case 'recordPayment':
                  handleRecordPayment(record);
                  break;
                default:
                  break;
              }
              // else if (key === '2')handleCloseTask
            },
          }}
          trigger={['click']}
        >
          <EllipsisOutlined
            style={{ cursor: 'pointer', fontSize: '24px' }}
            onClick={(e) => e.preventDefault()}
          />
        </Dropdown>
      ),
    },
  ];

  const dispatch = useDispatch();

  const handelDataTableLoad = (pagination) => {
    const options = { page: pagination.current || 1, items: pagination.pageSize || 10 };
    dispatch(erp.list({ entity, options }));
  };

  const dispatcher = () => {
    dispatch(erp.list({ entity }));
  };

  useEffect(() => {
    const controller = new AbortController();
    dispatcher();
    return () => {
      controller.abort();
    };
  }, []);

  const { expandedRowData, tableColumns, tableHeader } = useResponsiveTable(
    dataTableColumns,
    items
  );

  {/* progrsh bar */ }

  {/* progrsh bar */ }

  {/* email split code onlye show name */ }

  const getEmailName = (email) => {
    if (!email) return '';
    const parts = email.split('@');
    return parts[0];
  };
  // Helper function to calculate progress percentage
  // Helper function to calculate progress percentage
  const calculateProgress = (value, target) => {
    const targetAmount = 2000000; // 2 crore
    if (!target || target === 0) return 0;
    const progress = (value / targetAmount) * 100;
    return Math.min(Math.floor(progress), 100); // Limit progress to 100%
  };
  {/* email split code onlye show name */ }
  return (
    <>

      {/*filter components section  */}
      <div className='mb-10 -mt-10 relative'>
        <div>
          <GrPowerReset onClick={resetData} className='float-right -mr-4 text-xl font-thin text-red-600 cursor-pointer' title='Reset All ' />
        </div>
      </div>
      <div className='flex justify-items-start items-center mb-10 gap-3 -mt-4'>
        <Select className='w-72 shadow h-10'
          value={selectedInstitute}
          onChange={handleInstituteChange}
          placeholder="Select Institute Name"
        >
          <Select.Option value=''>Select Institute Name</Select.Option>
          {institutes.map((option) => (
            <Select.Option key={option} value={option}>
              {option}
            </Select.Option>
          ))}
        </Select>

        <Select className='w-72 shadow h-10'
          value={selectedUniversity}
          onChange={handleUniversityChange}
        >
          <Select.Option value=''>Select University Name</Select.Option>
          {universities.map((option) => (
            <Select.Option key={option} value={option}>
              {option}
            </Select.Option>
          ))}
        </Select>

        <Select className='w-72 shadow h-10'
          value={selectedCounselor}
          onChange={handleCounselorChange}
        >
          <Select.Option value=''>Select Counselor</Select.Option>
          {counselors.map((email) => (
            <Select.Option key={email} value={email}>
              {getEmailName(email)}
            </Select.Option>
          ))}
        </Select>
        <Select className='w-72 shadow h-10'
          value={selectedPayment}
          onChange={handlePaymentChange}
        >
          <Select.Option value=''>Select Payment Type</Select.Option>
          {paymentType.map((payment) => (
            <Select.Option key={payment} value={payment}>
              {payment}
            </Select.Option>
          ))}
        </Select>
        <Input className='text-sm font-thin w-72 shadow h-10'
          type='date'
          onChange={handleDateChange}
          value={selectedDate}
          style={{ width: '140px', marginRight: '16px', textTransform: 'uppercase', }}
        />



      </div>
      {/*filter components section  */}

      {/* payments component card code */}
      <div ref={tableHeader}>
        <div className='mb-24'>
          {entity === 'payment' && (
            <div className='mb-10 flex gap-4'>
              <Card className="w-1/3 shadow-lg">
                <div className='flex justify-between'>
                  <div>
                    <CardContent orientation="horizontal">
                      <CardContent>
                        <Typography className="text-gray-500">Total Course Fee</Typography>
                        <Typography level="h3" className="text-green-500">₹ {filteredPaymentData?.total_course_fee || 0}</Typography>
                      </CardContent>
                    </CardContent>
                  </div>
                  <div>
                    <CircularProgress size="lg" determinate value={calculateProgress(filteredPaymentData.total_course_fee, filteredPaymentData.totalPaymentAmount?.totalCourseFee)}>
                      <SvgIcon>
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          fill="none"
                          viewBox="0 0 24 24"
                          strokeWidth={1.5}
                          stroke="currentColor"
                        >
                          <path
                            strokeLinecap="round"
                            strokeLinejoin="round"
                            d="M2.25 18L9 11.25l4.306 4.307a11.95 11.95 0 015.814-5.519l2.74-1.22m0 0l-5.94-2.28m5.94 2.28l-2.28 5.941"
                          />
                        </svg>
                      </SvgIcon>
                    </CircularProgress>
                  </div>
                </div>
                <Progress percent={calculateProgress(filteredPaymentData.total_course_fee, filteredPaymentData.totalPaymentAmount?.totalCourseFee)} status="active" className='mt-3' />
              </Card>
              <Card className="w-1/3 shadow-lg">
                <div className='flex justify-between'>
                  <div>
                    <CardContent orientation="horizontal">
                      <CardContent>
                        <Typography className="text-gray-500">Total Paid Amount</Typography>
                        <Typography level="h3" className="text-blue-500">₹ {filteredPaymentData.total_paid_amount || 0}</Typography>
                      </CardContent>
                    </CardContent>
                  </div>
                  <div>
                    <CircularProgress size="lg" determinate value={calculateProgress(filteredPaymentData.total_paid_amount, filteredPaymentData.totalPaymentAmount?.totalPaidAmount)}>
                      <SvgIcon>
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          fill="none"
                          viewBox="0 0 24 24"
                          strokeWidth={1.5}
                          stroke="currentColor"
                        >
                          <path
                            strokeLinecap="round"
                            strokeLinejoin="round"
                            d="M2.25 18L9 11.25l4.306 4.307a11.95 11.95 0 015.814-5.519l2.74-1.22m0 0l-5.94-2.28m5.94 2.28l-2.28 5.941"
                          />
                        </svg>
                      </SvgIcon>
                    </CircularProgress>
                  </div>
                </div>
                <Progress percent={calculateProgress(filteredPaymentData.total_paid_amount, filteredPaymentData.totalPaymentAmount?.totalPaidAmount)} status="active" className='mt-3' />
              </Card>
              <Card className="w-1/3 shadow-lg">
                <div className='flex justify-between'>
                  <div>
                    <CardContent orientation="horizontal">
                      <CardContent>
                        <Typography className="text-gray-500">Due Amount</Typography>
                        <Typography level="h3" className="text-red-500">₹ {filteredPaymentData.due_amount || 0}</Typography>
                      </CardContent>
                    </CardContent>
                  </div>
                  <div>
                    <CircularProgress size="lg" determinate value={calculateProgress(filteredPaymentData.due_amount, filteredPaymentData.totalPaymentAmount?.totalCourseFee)}>
                      <SvgIcon>
                        <svg
                          xmlns="http://www.w3.org/2000/svg"
                          fill="none"
                          viewBox="0 0 24 24"
                          strokeWidth={1.5}
                          stroke="currentColor"
                        >
                          <path
                            strokeLinecap="round"
                            strokeLinejoin="round"
                            d="M2.25 18L9 11.25l4.306 4.307a11.95 11.95 0 015.814-5.519l2.74-1.22m0 0l-5.94-2.28m5.94 2.28l-2.28 5.941"
                          />
                        </svg>
                      </SvgIcon>
                    </CircularProgress>
                  </div>
                </div>
                <Progress percent={calculateProgress(filteredPaymentData.due_amount, filteredPaymentData.totalPaymentAmount?.totalCourseFee)} status="active" className='mt-3' />
              </Card>
            </div>
          )}
        </div>
        {/* payments component card code */}



        {/* table components code */}
        <PageHeader
          ghost={true}
          style={{
            padding: '20px 0px',
          }}
        >
          <div className='flex justify-between items-center'>
            <div>
              <Search placeholder="input search text" allowClear style={{ width: 200 }} />
            </div>
            <div className='flex items-center gap-4'>
              <Button onClick={handelDataTableLoad} key={`${uniqueId()}`} icon={<AiOutlineExport />} className='flex items-center bg-transparent text-gray-500 hover:text-black p-4 h-6'>
                {translate('Export')}
              </Button>
              <AddNewItem config={config} key={`${uniqueId()}`} hasCreate={create} />
            </div>

          </div></PageHeader>
      </div>
      <Table
        columns={tableColumns}
        rowKey={(item) => item._id}
        dataSource={dataSource}
        pagination={pagination}
        loading={listIsLoading}
        onChange={handelDataTableLoad}
        expandable={
          expandedRowData.length
            ? {
              expandedRowRender: (record) => (
                <Descriptions title="" bordered column={1}>
                  {expandedRowData.map((item, index) => {
                    return (
                      <Descriptions.Item key={index} label={item.title}>
                        {item.render?.(record[item.dataIndex])?.children
                          ? item.render?.(record[item.dataIndex])?.children
                          : item.render?.(record[item.dataIndex])
                            ? item.render?.(record[item.dataIndex])
                            : Array.isArray(item.dataIndex)
                              ? record[item.dataIndex[0]]?.[item.dataIndex[1]]
                              : record[item.dataIndex]}
                      </Descriptions.Item>
                    );
                  })}
                </Descriptions>
              ),
            }
            : null
        }
      />
      {/* table components code */}
    </>
  );
}


