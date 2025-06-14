import React, { useState, useEffect } from 'react';

// Tailwind CSS is assumed to be available in the environment.

// Helper function to convert a File object to a Base64 string
const fileToBase64 = (file) => {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
    });
};

export default function App() {
    // State for landlord and tenant information
    const [landlordInfo, setLandlordInfo] = useState({
        name: '',
        address: '',
        phone: '',
        email: ''
    });

    const [tenantInfo, setTenantInfo] = useState({
        name: '',
        phone: '',
        email: ''
    });

    // State for selected US state
    const [selectedState, setSelectedState] = useState('');

    // State for room damage descriptions and photos
    const [roomDamages, setRoomDamages] = useState([
        { id: 1, roomName: '', description: '', photos: [] }
    ]);
    const [nextRoomId, setNextRoomId] = useState(2);

    // State for document generation loading
    const [isGenerating, setIsGenerating] = useState(false);
    const [showDownloadModal, setShowDownloadModal] = useState(false);
    const [generatedHtmlContent, setGeneratedHtmlContent] = useState('');

    // Predefined list of US states for the dropdown
    const usStates = [
        "Alabama", "Alaska", "Arizona", "Arkansas", "California", "Colorado", "Connecticut", "Delaware", "Florida",
        "Georgia", "Hawaii", "Idaho", "Illinois", "Indiana", "Iowa", "Kansas", "Kentucky", "Louisiana", "Maine",
        "Maryland", "Massachusetts", "Michigan", "Minnesota", "Mississippi", "Missouri", "Montana", "Nebraska",
        "Nevada", "New Hampshire", "New Jersey", "New Mexico", "New York", "North Carolina", "North Dakota", "Ohio",
        "Oklahoma", "Oregon", "Pennsylvania", "Rhode Island", "South Carolina", "South Dakota", "Tennessee", "Texas",
        "Utah", "Vermont", "Virginia", "Washington", "West Virginia", "Wisconsin", "Wyoming"
    ];

    // Handle input changes for landlord and tenant info
    const handleLandlordChange = (e) => {
        const { name, value } = e.target;
        setLandlordInfo(prev => ({ ...prev, [name]: value }));
    };

    const handleTenantChange = (e) => {
        const { name, value } = e.target;
        setTenantInfo(prev => ({ ...prev, [name]: value }));
    };

    // Handle room damage input changes
    const handleRoomChange = (id, e) => {
        const { name, value } = e.target;
        setRoomDamages(prev =>
            prev.map(room => (room.id === id ? { ...room, [name]: value } : room))
        );
    };

    // Handle photo selection for a room
    const handlePhotoChange = async (id, e) => {
        const files = Array.from(e.target.files);
        setRoomDamages(prev =>
            prev.map(room => (room.id === id ? { ...room, photos: files } : room))
        );
    };

    // Add a new room entry
    const addRoom = () => {
        setRoomDamages(prev => [
            ...prev,
            { id: nextRoomId, roomName: '', description: '', photos: [] }
        ]);
        setNextRoomId(prev => prev + 1);
    };

    // Remove a room entry
    const removeRoom = (id) => {
        setRoomDamages(prev => prev.filter(room => room.id !== id));
    };

    // Generate the document content (HTML string)
    const generateDocumentContent = async () => {
        let photosHtml = '';
        for (const room of roomDamages) {
            let roomPhotosHtml = '';
            if (room.photos && room.photos.length > 0) {
                for (const photo of room.photos) {
                    try {
                        const base64 = await fileToBase64(photo);
                        roomPhotosHtml += `
                            <div class="mb-2 p-2 border border-gray-200 rounded-md">
                                <img src="${base64}" alt="${room.roomName} Damage" class="w-full h-auto rounded-md object-cover mb-2" />
                                <p class="text-sm text-gray-600">File Name: ${photo.name}</p>
                            </div>
                        `;
                    } catch (error) {
                        console.error("Error loading photo:", error);
                        roomPhotosHtml += `<p class="text-red-500">Could not load photo: ${photo.name}</p>`;
                    }
                }
            } else {
                roomPhotosHtml = '<p class="text-gray-500 italic">No photos added.</p>';
            }

            photosHtml += `
                <div class="mb-6 p-4 border border-gray-300 rounded-lg shadow-sm">
                    <h4 class="text-lg font-semibold text-gray-800 mb-2">${room.roomName || 'Unknown Room'}</h4>
                    <p class="text-gray-700 mb-2"><strong>Description:</strong> ${room.description || 'No description entered.'}</p>
                    <p class="text-gray-700 mb-2"><strong>Photos:</strong></p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-4">
                        ${roomPhotosHtml}
                    </div>
                </div>
            `;
        }

        const html = `
            <div class="font-sans p-6 bg-white text-gray-800">
                <h1 class="text-3xl font-bold text-center text-indigo-700 mb-6">Property Handover Checklist</h1>
                <p class="text-center text-gray-600 mb-8">(${selectedState ? selectedState : 'Selected State'})</p>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-8 mb-8">
                    <div class="p-5 border border-indigo-200 rounded-lg shadow-md bg-indigo-50">
                        <h2 class="text-xl font-semibold text-indigo-600 mb-4">Landlord Information</h2>
                        <p class="mb-2"><strong>Full Name/Organization Name:</strong> ${landlordInfo.name}</p>
                        <p class="mb-2"><strong>Address:</strong> ${landlordInfo.address}</p>
                        <p class="mb-2"><strong>Phone:</strong> ${landlordInfo.phone}</p>
                        <p class="mb-2"><strong>Email:</strong> ${landlordInfo.email}</p>
                    </div>

                    <div class="p-5 border border-green-200 rounded-lg shadow-md bg-green-50">
                        <h2 class="text-xl font-semibold text-green-600 mb-4">Tenant Information</h2>
                        <p class="mb-2"><strong>Full Name:</strong> ${tenantInfo.name}</p>
                        <p class="mb-2"><strong>Phone:</strong> ${tenantInfo.phone}</p>
                        <p class="mb-2"><strong>Email:</strong> ${tenantInfo.email}</p>
                    </div>
                </div>

                <div class="p-6 border border-gray-200 rounded-lg shadow-md bg-gray-50 mb-8">
                    <h2 class="text-xl font-semibold text-gray-700 mb-4">Property Condition and Existing Damages</h2>
                    <p class="mb-4 text-gray-600 italic">
                        This section details the current condition and any existing damages of the property at the start of the lease.
                        This document reflects the property's condition upon the tenant's receipt and serves as a reference for resolving potential disputes in the future.
                    </p>
                    ${photosHtml}
                </div>

                <div class="p-6 border border-yellow-200 rounded-lg shadow-md bg-yellow-50 mb-8">
                    <h2 class="text-xl font-semibold text-yellow-700 mb-4">General Provisions and State-Specific Notes</h2>
                    <ul class="list-disc list-inside text-gray-700">
                        <li class="mb-2">The tenant is obliged to use the property as stated in the lease agreement.</li>
                        <li class="mb-2">The tenant is responsible for remedying any new damages (excluding normal wear and tear) that occur to the property during the lease term.</li>
                        <li class="mb-2">Upon return of the property, if new damages not specified in this checklist are found, repair costs may be deducted from the security deposit.</li>
                        <li class="mb-2">This document shall be interpreted in accordance with the rental and property laws of ${selectedState ? selectedState : 'the relevant state'}.</li>
                        <li class="mb-2 font-bold text-red-700">**State-Specific Note: (In a real application, this field would automatically be populated with important clauses from the selected state's rental laws. This is a placeholder for demonstration purposes.)**</li>
                    </ul>
                </div>

                <div class="mt-8 pt-6 border-t border-gray-300 text-center text-gray-600 text-sm">
                    <p class="mb-2">This document was generated on ${new Date().toLocaleDateString('en-US')}.</p>
                    <p class="italic">The landlord and tenant confirm the accuracy of the above information and agree to have received and handed over the property under the conditions stated in this document.</p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-8 mt-8">
                        <div>
                            <p class="mb-4">_________________________</p>
                            <p>Landlord Signature</p>
                            <p>${landlordInfo.name}</p>
                        </div>
                        <div>
                            <p class="mb-4">_________________________</p>
                            <p>Tenant Signature</p>
                            <p>${tenantInfo.name}</p>
                        </div>
                    </div>
                </div>
            </div>
        `;
        setGeneratedHtmlContent(html);
        return html;
    };

    // Handle PDF download
    const downloadPdf = async () => {
        setIsGenerating(true);
        try {
            const contentHtml = await generateDocumentContent();
            const element = document.createElement('div');
            element.innerHTML = contentHtml;
            element.style.width = '210mm'; // A4 width
            element.style.padding = '10mm'; // Add some padding for better printing
            element.style.backgroundColor = '#fff'; // Ensure white background

            // Append element temporarily to body for html2canvas to render
            document.body.appendChild(element);

            // Access html2canvas from the global window object
            const canvas = await window.html2canvas(element, {
                scale: 2, // Increase scale for better quality
                useCORS: true, // Important for images loaded from data URLs
                logging: true,
                allowTaint: true, // Allow tainting for images if they are base64
                scrollX: 0,
                scrollY: -window.scrollY // Capture from top of the page
            });

            const imgData = canvas.toDataURL('image/png');
            // Access jsPDF from the global window object
            const pdf = new window.jspdf.jsPDF('p', 'mm', 'a4'); // 'p' for portrait, 'mm' for millimeters, 'a4' size
            const imgWidth = 210; // A4 width in mm
            const pageHeight = 297; // A4 height in mm
            const imgHeight = canvas.height * imgWidth / canvas.width;
            let heightLeft = imgHeight;

            let position = 0;

            pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
            heightLeft -= pageHeight;

            while (heightLeft >= 0) {
                position = heightLeft - imgHeight;
                pdf.addPage();
                pdf.addImage(imgData, 'PNG', 0, position, imgWidth, imgHeight);
                heightLeft -= pageHeight;
            }

            pdf.save('property_handover_checklist.pdf'); // Updated file name

            // Remove the temporary element
            document.body.removeChild(element);
        } catch (error) {
            console.error("Error generating PDF:", error);
            alert("An error occurred while generating the PDF. Please try again.");
        } finally {
            setIsGenerating(false);
            setShowDownloadModal(false); // Close modal after download attempt
        }
    };

    // Handle Word download (simplified: generates HTML that can be saved as .doc)
    const downloadWord = async () => {
        const contentHtml = await generateDocumentContent();
        const header = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'><head><meta charset='utf-8'><title>Property Handover Checklist</title></head><body>`; // Updated title
        const footer = `</body></html>`;
        const sourceHTML = header + contentHtml + footer;

        const blob = new Blob([sourceHTML], { type: 'application/msword' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = 'property_handover_checklist.doc'; // Updated file name
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        setShowDownloadModal(false); // Close modal after download attempt
    };

    // Trigger document generation and show modal
    const handleGenerateClick = async () => {
        setIsGenerating(true);
        try {
            await generateDocumentContent(); // Ensure content is generated
            setShowDownloadModal(true);
        } catch (error) {
            console.error("Error generating document content:", error);
            alert("An error occurred while generating the document content.");
        } finally {
            setIsGenerating(false);
        }
    };

    return (
        <div className="min-h-screen bg-gray-100 p-4 sm:p-6 lg:p-8 font-inter antialiased">
            <div className="max-w-4xl mx-auto bg-white p-6 sm:p-8 rounded-2xl shadow-xl border border-gray-200">
                <h1 className="text-3xl sm:text-4xl font-extrabold text-center text-indigo-800 mb-8 pb-4 border-b-2 border-indigo-200">
                    Property Handover Checklist Generator
                </h1>

                {/* Landlord Information */}
                <section className="mb-8 p-6 bg-indigo-50 rounded-xl shadow-inner border border-indigo-100">
                    <h2 className="text-2xl font-bold text-indigo-700 mb-5 flex items-center">
                        <svg className="w-7 h-7 mr-3 text-indigo-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                            <path fillRule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clipRule="evenodd"></path>
                        </svg>
                        1. Landlord Information
                    </h2>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div className="mb-4">
                            <label htmlFor="landlordName" className="block text-gray-700 text-sm font-semibold mb-2">Full Name/Organization Name:</label>
                            <input
                                type="text"
                                id="landlordName"
                                name="name"
                                value={landlordInfo.name}
                                onChange={handleLandlordChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400 transition duration-200 ease-in-out"
                                placeholder="E.g., Jane Doe"
                                required
                            />
                        </div>
                        <div className="mb-4">
                            <label htmlFor="landlordAddress" className="block text-gray-700 text-sm font-semibold mb-2">Property Address:</label>
                            <input
                                type="text"
                                id="landlordAddress"
                                name="address"
                                value={landlordInfo.address}
                                onChange={handleLandlordChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400 transition duration-200 ease-in-out"
                                placeholder="E.g., 123 Main St, Anytown, CA 90210"
                                required
                            />
                        </div>
                        <div className="mb-4">
                            <label htmlFor="landlordPhone" className="block text-gray-700 text-sm font-semibold mb-2">Phone:</label>
                            <input
                                type="tel"
                                id="landlordPhone"
                                name="phone"
                                value={landlordInfo.phone}
                                onChange={handleLandlordChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400 transition duration-200 ease-in-out"
                                placeholder="E.g., (555) 123-4567"
                            />
                        </div>
                        <div className="mb-4">
                            <label htmlFor="landlordEmail" className="block text-gray-700 text-sm font-semibold mb-2">Email:</label>
                            <input
                                type="email"
                                id="landlordEmail"
                                name="email"
                                value={landlordInfo.email}
                                onChange={handleLandlordChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-indigo-300 focus:border-indigo-400 transition duration-200 ease-in-out"
                                placeholder="E.g., jane.doe@example.com"
                            />
                        </div>
                    </div>
                </section>

                {/* Tenant Information */}
                <section className="mb-8 p-6 bg-green-50 rounded-xl shadow-inner border border-green-100">
                    <h2 className="text-2xl font-bold text-green-700 mb-5 flex items-center">
                        <svg className="w-7 h-7 mr-3 text-green-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                            <path fillRule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clipRule="evenodd"></path>
                        </svg>
                        2. Tenant Information
                    </h2>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                        <div className="mb-4">
                            <label htmlFor="tenantName" className="block text-gray-700 text-sm font-semibold mb-2">Full Name:</label>
                            <input
                                type="text"
                                id="tenantName"
                                name="name"
                                value={tenantInfo.name}
                                onChange={handleTenantChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-green-300 focus:border-green-400 transition duration-200 ease-in-out"
                                placeholder="E.g., John Smith"
                                required
                            />
                        </div>
                        <div className="mb-4">
                            <label htmlFor="tenantPhone" className="block text-gray-700 text-sm font-semibold mb-2">Phone:</label>
                            <input
                                type="tel"
                                id="tenantPhone"
                                name="phone"
                                value={tenantInfo.phone}
                                onChange={handleTenantChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-green-300 focus:border-green-400 transition duration-200 ease-in-out"
                                placeholder="E.g., (555) 987-6543"
                            />
                        </div>
                        <div className="mb-4">
                            <label htmlFor="tenantEmail" className="block text-gray-700 text-sm font-semibold mb-2">Email:</label>
                            <input
                                type="email"
                                id="tenantEmail"
                                name="email"
                                value={tenantInfo.email}
                                onChange={handleTenantChange}
                                className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-green-300 focus:border-green-400 transition duration-200 ease-in-out"
                                placeholder="E.g., john.smith@example.com"
                            />
                        </div>
                    </div>
                </section>

                {/* State Selection */}
                <section className="mb-8 p-6 bg-blue-50 rounded-xl shadow-inner border border-blue-100">
                    <h2 className="text-2xl font-bold text-blue-700 mb-5 flex items-center">
                        <svg className="w-7 h-7 mr-3 text-blue-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                            <path fillRule="evenodd" d="M5.05 4.05a7 7 0 119.9 9.9L10 18.9l-4.95-4.95a7 7 0 010-9.9zM10 11a2 2 0 100-4 2 2 0 000 4z" clipRule="evenodd"></path>
                        </svg>
                        3. State Selection
                    </h2>
                    <div className="mb-4">
                        <label htmlFor="stateSelect" className="block text-gray-700 text-sm font-semibold mb-2">Select State for the Document:</label>
                        <select
                            id="stateSelect"
                            value={selectedState}
                            onChange={(e) => setSelectedState(e.target.value)}
                            className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-300 focus:border-blue-400 transition duration-200 ease-in-out"
                            required
                        >
                            <option value="">Please Select a State</option>
                            {usStates.map(state => (
                                <option key={state} value={state}>{state}</option>
                            ))}
                        </select>
                    </div>
                </section>

                {/* Property Condition and Existing Damages */}
                <section className="mb-8 p-6 bg-gray-50 rounded-xl shadow-inner border border-gray-100">
                    <h2 className="text-2xl font-bold text-gray-700 mb-5 flex items-center">
                        <svg className="w-7 h-7 mr-3 text-gray-600" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                            <path fillRule="evenodd" d="M5 3a2 2 0 00-2 2v10a2 2 0 002 2h10a2 2 0 002-2V5a2 2 0 00-2-2H5zm0 2h10v7a2 2 0 01-2 2H7a2 2 0 01-2-2V5zm10 4a1 1 0 100 2 1 1 0 000-2z" clipRule="evenodd"></path>
                        </svg>
                        4. Property Condition and Existing Damages
                    </h2>
                    <p className="text-gray-600 mb-6 text-sm">
                        Please describe the current condition and any existing damages for each room or area of the property. You can also upload photos to visually document the condition.
                    </p>

                    {roomDamages.map(room => (
                        <div key={room.id} className="p-5 mb-6 border border-gray-200 rounded-lg bg-white shadow-sm relative">
                            {roomDamages.length > 1 && (
                                <button
                                    onClick={() => removeRoom(room.id)}
                                    className="absolute top-3 right-3 bg-red-100 text-red-600 hover:bg-red-200 p-2 rounded-full text-xs font-semibold transition-colors duration-200"
                                    title="Remove Room"
                                >
                                    <svg className="w-4 h-4" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                        <path fillRule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm6 0a1 1 0 11-2 0v6a1 1 0 112 0V8z" clipRule="evenodd"></path>
                                    </svg>
                                </button>
                            )}
                            <div className="mb-4">
                                <label htmlFor={`roomName-${room.id}`} className="block text-gray-700 text-sm font-semibold mb-2">Room/Area Name:</label>
                                <input
                                    type="text"
                                    id={`roomName-${room.id}`}
                                    name="roomName"
                                    value={room.roomName}
                                    onChange={(e) => handleRoomChange(room.id, e)}
                                    className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-gray-300 focus:border-gray-400 transition duration-200 ease-in-out"
                                    placeholder="E.g., Living Room, Kitchen, Bathroom"
                                    required
                                />
                            </div>
                            <div className="mb-4">
                                <label htmlFor={`description-${room.id}`} className="block text-gray-700 text-sm font-semibold mb-2">Damage Description (Optional):</label>
                                <textarea
                                    id={`description-${room.id}`}
                                    name="description"
                                    value={room.description}
                                    onChange={(e) => handleRoomChange(room.id, e)}
                                    rows="3"
                                    className="shadow-sm appearance-none border rounded-md w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-gray-300 focus:border-gray-400 transition duration-200 ease-in-out"
                                    placeholder="E.g., Slight scratches on walls, small stain on kitchen counter."
                                ></textarea>
                            </div>
                            <div className="mb-4">
                                <label htmlFor={`photos-${room.id}`} className="block text-gray-700 text-sm font-semibold mb-2">Upload Photos (For Existing Damages):</label>
                                <input
                                    type="file"
                                    id={`photos-${room.id}`}
                                    name="photos"
                                    multiple
                                    accept="image/*"
                                    onChange={(e) => handlePhotoChange(room.id, e)}
                                    className="block w-full text-sm text-gray-700 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 transition duration-200 ease-in-out"
                                />
                                <div className="mt-2 flex flex-wrap gap-2">
                                    {room.photos && room.photos.map((photo, index) => (
                                        <img
                                            key={index}
                                            src={URL.createObjectURL(photo)}
                                            alt={`Photo ${index + 1}`}
                                            className="w-24 h-24 object-cover rounded-md border border-gray-200 shadow-sm"
                                        />
                                    ))}
                                </div>
                            </div>
                        </div>
                    ))}
                    <button
                        onClick={addRoom}
                        className="w-full bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-3 px-4 rounded-xl shadow-lg hover:shadow-xl transition-all duration-300 ease-in-out flex items-center justify-center text-lg mt-6"
                    >
                        <svg className="w-6 h-6 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
                        </svg>
                        Add New Room/Area
                    </button>
                </section>

                {/* Generate Document Button */}
                <div className="mt-10 text-center">
                    <button
                        onClick={handleGenerateClick}
                        disabled={isGenerating || !landlordInfo.name || !tenantInfo.name || !selectedState || roomDamages.some(r => !r.roomName)}
                        className={`bg-purple-600 hover:bg-purple-700 text-white font-bold py-4 px-8 rounded-full shadow-lg text-xl transition-all duration-300 ease-in-out transform hover:scale-105
                            ${(isGenerating || !landlordInfo.name || !tenantInfo.name || !selectedState || roomDamages.some(r => !r.roomName)) ? 'opacity-50 cursor-not-allowed' : ''}`}
                    >
                        {isGenerating ? 'Generating Document...' : 'Generate and Download Document'}
                    </button>
                    {(!landlordInfo.name || !tenantInfo.name || !selectedState || roomDamages.some(r => !r.roomName)) && (
                        <p className="text-red-500 text-sm mt-3">Please fill in all required fields (Landlord Name, Tenant Name, State Selection, and at least one Room Name).</p>
                    )}
                </div>

                {/* Download Options Modal */}
                {showDownloadModal && (
                    <div className="fixed inset-0 bg-gray-600 bg-opacity-75 flex items-center justify-center p-4 z-50">
                        <div className="bg-white p-8 rounded-xl shadow-2xl text-center max-w-sm w-full border border-gray-200">
                            <h3 className="text-2xl font-bold text-gray-800 mb-6">Document Ready!</h3>
                            <p className="text-gray-600 mb-8">Which format would you like to download?</p>
                            <div className="flex flex-col gap-4">
                                <button
                                    onClick={downloadPdf}
                                    className="bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg shadow-md transition-colors duration-200 flex items-center justify-center text-lg"
                                >
                                    <svg className="w-6 h-6 mr-2" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                        <path fillRule="evenodd" d="M4 4a2 2 0 012-2h4.586A2 2 0 0113 3.414L16.586 7A2 2 0 0117 8.414V16a2 2 0 01-2 2H6a2 2 0 01-2-2V4zm2 0v4a2 2 0 002 2h4v5a1 1 0 01-1 1H7a1 1 0 01-1-1v-7z" clipRule="evenodd"></path>
                                    </svg>
                                    Download as PDF
                                </button>
                                <button
                                    onClick={downloadWord}
                                    className="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-6 rounded-lg shadow-md transition-colors duration-200 flex items-center justify-center text-lg"
                                >
                                    <svg className="w-6 h-6 mr-2" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                        <path d="M9 2a1 1 0 000 2h2a1 1 0 100-2H9z"></path>
                                        <path fillRule="evenodd" d="M4 5a2 2 0 012-2h4.586A2 2 0 0113 3.414L16.586 7A2 2 0 0117 8.414V16a2 2 0 01-2 2H6a2 2 0 01-2-2V5zm3 8V7h4v6H7zm6-4H7V7h6v2zm-2-2H9v2h2V9z" clipRule="evenodd"></path>
                                    </svg>
                                    Download as Word
                                </button>
                                <button
                                    onClick={() => setShowDownloadModal(false)}
                                    className="mt-4 text-gray-500 hover:text-gray-700 font-semibold py-2 px-4 rounded-lg"
                                >
                                    Cancel
                                </button>
                            </div>
                        </div>
                    </div>
                )}
            </div>
        </div>
    );
}
